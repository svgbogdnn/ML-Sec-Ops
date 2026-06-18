# ЛР 4. Недостатки в python-коде. Механизмы безопасности в django

## Часть 1

### Фрагмент кода 1

```python
#!/usr/bin/env python3
from flask import Flask

from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')
```

**Недостаток:**

Секретный ключ (SECRET_KEY) захардкожен в исходном коде приложения

**Меры устранения:**

Необходимо вынести секретный ключ в переменные окружения или использовать менеджеры секретов, чтобы исключить его утечку и повысить безопасность приложения


### Фрагмент кода 2

```python
from flask_wtf import Form  
from wtforms import TextField

class LoginForm(Form):  
	username = TextField('username')  
	password = TextField('password')
```

**Недостаток:**  
Отсутствует валидация пользовательского ввода — поля формы принимают любые данные без проверки

**Меры устранения:**  
Следует добавить валидацию (ограничение длины, формат данных), использовать PasswordField для пароля и встроенные механизмы защиты

### Фрагмент кода 3

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

**Недостаток:**  
Некорректная обработка исключений — при возникновении ошибки пользователю возвращается сообщение об успешном выполнении операции даже при ошибке

**Меры устранения:**  
Необходимо обрабатывать ошибки корректно, логировать исключения и возвращать пользователю соответствующий результат (ошибку, а не успех)

---

## Часть 2

### 1. Какие наиболее распространенные атаки предотвращает Django?

Django имеет встроенные механизмы защиты от следующих атак:

- Cross-Site Scripting (XSS) — благодаря автоматическому экранированию шаблонов
- Cross-Site Request Forgery (CSRF) — через CSRF-токены
- SQL-инъекции — за счёт ORM, который использует параметризованные запросы
- Clickjacking — с помощью заголовка X-Frame-Options
- Session Hijacking — через безопасную работу с cookies (HttpOnly, Secure)

### 2. Управление пользовательскими сессиями в Django

Django использует встроенную систему управления сессиями, которая позволяет хранить состояние пользователя между HTTP-запросами

Основные особенности:

- Каждому пользователю назначается уникальный session ID
- Session ID хранится в cookies браузера
- Данные сессии хранятся на сервере (в базе данных, кеше или файлах)

Механизм работы:

1. При первом запросе создаётся сессия и генерируется session ID
2. Session ID отправляется пользователю в cookie
3. При последующих запросах браузер отправляет этот cookie обратно серверу
4. Django использует session ID для получения данных сессии

Меры безопасности:

- cookies могут быть помечены как HttpOnly (защита от XSS)
- cookies могут быть Secure (передаются только по HTTPS)
- поддержка автоматического истечения сессий
- защита от подделки сессий за счёт SECRET_KEY
