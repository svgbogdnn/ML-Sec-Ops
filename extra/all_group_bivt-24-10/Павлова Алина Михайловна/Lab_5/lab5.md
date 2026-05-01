# Лабораторная работа №5: Поиск уязвимостей в Python-коде и безопасность Django

---

## Часть I: Анализ фрагментов кода

### Фрагмент кода 1: Flask с SECRET_KEY

```python
#!/usr/bin/env python3
from flask import Flask
from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')
```

**Недостаток:**  
Секретный ключ (`SECRET_KEY`) захардкожен прямо в исходном коде приложения.

**Риски:**
- При утечке исходного кода злоумышленник получает ключ
- Возможна подделка сессионных кук и CSRF-токенов
- Невозможность быстрой ротации ключа без изменения кода и деплоя

**Рекомендации по устранению:**
1. Хранить `SECRET_KEY` в переменной окружения:
   ```python
   import os
   app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
   ```
2. Генерировать ключ через `secrets.token_hex(32)`
3. Использовать `.env` файл (добавить в `.gitignore`)
4. Для production — использовать секрет-менеджер (HashiCorp Vault, AWS Secrets Manager)

---

### Фрагмент кода 2: Форма Flask-WTF

```python
from flask_wtf import Form  
from wtforms import TextField

class LoginForm(Form):  
    username = TextField('username')  
    password = TextField('password')
```

**Недостатки:**
1. Используется устаревший `TextField` вместо `StringField`
2. Отсутствуют валидаторы (`DataRequired`, `Length` и др.)
3. Поле `password` не маскируется при вводе
4. Нет явного включения CSRF-защиты

**Рекомендации по устранению:**
1. Использовать актуальные классы и валидаторы:
   ```python
   from flask_wtf import FlaskForm
   from wtforms import StringField, PasswordField
   from wtforms.validators import DataRequired, Length

   class LoginForm(FlaskForm):
       username = StringField('username', validators=[DataRequired(), Length(min=3, max=50)])
       password = PasswordField('password', validators=[DataRequired(), Length(min=8)])
   ```
2. Включить CSRF-защиту: `app.config['WTF_CSRF_ENABLED'] = True`
3. Добавлять `{{ form.csrf_token }}` в HTML-формы
4. Добавить rate limiting для защиты от брутфорса

---

### Фрагмент кода 3: Обработка данных пользователя

```python
def post(self):
    username = self.get_argument('username')
    password = self.get_argument('password').encode('utf-8')
    email = self.get_argument('mail')

    try:
        username = username.lower()
        email = email.strip().lower()
        user = User({'username': username, 'password': password, 'email': email, 'date_joined': curtime()})
        user.validate()
        save_user(self.db_conn, user)
    except Exception, e:
        return self.render_template("success_create.html")
```

**Недостатки:**
1. Пароль не хешируется перед сохранением (`encode('utf-8')` ≠ хеширование)
2. Устаревший синтаксис `except Exception, e:` (Python 2)
3. При любой ошибке возвращается страница успеха — пользователь вводится в заблуждение
4. Нет валидации формата email и username
5. Возможна массовая ассигнмент: передача лишних полей в конструктор `User()`

**Рекомендации по устранению:**
1. Хешировать пароль:
   ```python
   import bcrypt
   password_hash = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
   ```
2. Использовать современный синтаксис: `except Exception as e:`
3. Логировать ошибки и возвращать корректные статусы:
   ```python
   except ValueError as e:
       return self.render_template("error.html", message=str(e)), 400
   ```
4. Валидировать входные данные (regex для email, длина username)
5. Использовать whitelist полей при создании объекта:
   ```python
   allowed_fields = ['username', 'password', 'email', 'date_joined']
   user_data = {k: v for k, v in input_data.items() if k in allowed_fields}
   ```

---

## Часть II: Модель безопасности Django

### Вопрос 1: Какие наиболее распространенные атаки предотвращает Django?

Django предоставляет встроенную защиту от следующих атак:

| Атака | Механизм защиты в Django |
|-------|-------------------------|
| **SQL Injection** | ORM автоматически экранирует параметры; raw SQL требует параметризованных запросов |
| **XSS (Cross-Site Scripting)** | Шаблонизатор авто-экранирует переменные: `{{ var }}` → `&lt;script&gt;` |
| **CSRF (Cross-Site Request Forgery)** | Middleware `CsrfViewMiddleware` требует токена в формах и AJAX-запросах |
| **Clickjacking** | `XFrameOptionsMiddleware` добавляет заголовок `X-Frame-Options: DENY` |
| **Session Hijacking** | Куки имеют флаги `HttpOnly` и `Secure` (при HTTPS) |
| **Password Brute-Force** | Хеш паролей через PBKDF2 с солью, 260 000 итераций по умолчанию |
| **Mass Assignment** | Forms/ModelForms требуют явного указания полей (`fields = [...]`) |
| **Open Redirect** | Функции `redirect()` проверяют, что целевой URL из разрешённых хостов |

> Все защиты включены по умолчанию при создании проекта через `django-admin startproject`.

---

### Вопрос 2: Как устроено управление пользовательскими сессиями в Django?

#### Архитектура сессий

1. **Создание сессии:**
   - При первом запросе или логине Django генерирует 32-символьный случайный `session_key`
   - Данные сессии сохраняются в бэкенд под этим ключом
   - Ключ отправляется браузеру в куке `sessionid`

2. **Бэкенды хранения:**
   | Бэкенд | Описание |
   |--------|----------|
   | `db` (по умолчанию) | Сессии в таблице `django_session` |
   | `cache` | Сессии в Redis/Memcached |
   | `cached_db` | Кэш + fallback в БД |
   | `signed_cookies` | Данные в подписанной куке (без хранения на сервере) |

3. **Безопасность кук (настройки по умолчанию):**
   ```python
   SESSION_COOKIE_HTTPONLY = True      # недоступно для JavaScript
   SESSION_COOKIE_SECURE = True        # только по HTTPS
   SESSION_COOKIE_SAMESITE = 'Lax'     # защита от CSRF
   SESSION_COOKIE_AGE = 1209600        # 2 недели в секундах
   ```

4. **Работа с сессиями в коде:**
   ```python
   # Запись
   request.session['user_id'] = user.id
   
   # Чтение
   user_id = request.session.get('user_id')
   
   # Удаление
   del request.session['user_id']
   request.session.flush()  # полная очистка
   ```

5. **Интеграция с аутентификацией:**
   - При `login(request, user)` Django создаёт **новую** сессию (защита от session fixation)
   - В сессию сохраняются: `_auth_user_id`, `_auth_user_backend`, `_auth_user_hash`
   - `AuthenticationMiddleware` при каждом запросе восстанавливает `request.user`

6. **Очистка устаревших сессий:**
   ```bash
   python manage.py clearsessions
   ```
   (Рекомендуется запускать по cron раз в день для бэкенда `db`)

