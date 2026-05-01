# ЛР 5. Недостатки в Python-коде и безопасность Django

## Цель работы

Цель работы: найти простые проблемы в python-коде и коротко разобрать, как Django помогает с безопасностью.

## Часть I. Разбор кода

### Фрагмент 1

```python
#!/usr/bin/env python3
from flask import Flask

from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')
```

Проблема: `SECRET_KEY` прямо лежит в коде. Это плохая идея, потому что код может попасть в git, архив, лог или просто к лишнему человеку.

Во Flask этот ключ нужен для подписи важных данных, например сессий. Если ключ утечет, то злоумышленник может пробовать подделывать такие данные. Это уже серьезный косяк.

Как исправить:

- не хранить секреты в коде;
- брать ключ из переменной окружения;
- делать разные ключи для dev и production;
- если ключ утек, сразу менять его.

Пример:

```python
import os
from flask import Flask

app = Flask("vuln_app")
app.config["SECRET_KEY"] = os.environ["FLASK_SECRET_KEY"]
```

### Фрагмент 2

```python
from flask_wtf import Form
from wtforms import TextField

class LoginForm(Form):
    username = TextField('username')
    password = TextField('password')
```

Проблема: пароль сделан как обычное текстовое поле. То есть пароль может быть виден на экране как простой текст. Для формы входа это прям не ок.

Еще нет проверок полей. Форма не проверяет, что логин и пароль заполнены и что они нормальной длины.

Как исправить:

- использовать `PasswordField` для пароля;
- добавить `DataRequired` и `Length`;
- использовать `FlaskForm`;
- не отключать CSRF-защиту.

Пример:

```python
from flask_wtf import FlaskForm
from wtforms import PasswordField, StringField
from wtforms.validators import DataRequired, Length

class LoginForm(FlaskForm):
    username = StringField("username", validators=[DataRequired(), Length(min=3, max=64)])
    password = PasswordField("password", validators=[DataRequired(), Length(min=8, max=128)])
```

### Фрагмент 3

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

Тут главный косяк: пароль не хешируется. Его просто переводят в UTF-8, но это не защита. Кодирование не делает пароль безопасным.

Пароли нельзя хранить как обычный текст или почти обычный текст. Надо хранить хеш пароля. Для этого подходят Argon2, bcrypt или PBKDF2. А MD5 и SHA-256 для паролей лучше не брать, они слишком быстро перебираются.

Еще одна проблема: `except Exception` ловит вообще все ошибки. И после ошибки пользователю показывается страница успеха. То есть аккаунт мог не создаться, но сайт скажет, что все норм.

Как исправить:

- хешировать пароль перед сохранением;
- не считать `.encode("utf-8")` защитой;
- ловить только ожидаемые ошибки;
- при ошибке показывать ошибку, а не страницу успеха;
- писать внутренние ошибки в лог.

Пример идеи:

```python
from passlib.hash import argon2

def post(self):
    username = self.get_argument("username").strip().lower()
    password = self.get_argument("password")
    email = self.get_argument("mail").strip().lower()

    password_hash = argon2.hash(password)

    try:
        user = User({
            "username": username,
            "password": password_hash,
            "email": email,
            "date_joined": curtime(),
        })
        user.validate()
        save_user(self.db_conn, user)
    except ValidationError:
        return self.render_template("create_user.html", error="Некорректные данные")

    return self.render_template("success_create.html")
```

## Часть II. Безопасность Django

### 1. От каких атак помогает Django

Django не делает проект магически безопасным, но у него есть нормальные встроенные защиты.

1. SQL injection.

   Если использовать Django ORM, то данные не склеиваются руками с SQL-запросом. Поэтому SQL-инъекцию сделать сложнее. Но если писать сырой SQL через строки, то можно снова накосячить.

2. XSS.

   В шаблонах Django переменные по умолчанию экранируются. Поэтому `<script>` от пользователя не должен выполниться в браузере. Но если поставить `safe` без проверки, защита может сломаться.

3. CSRF.

   Django проверяет CSRF-токен у POST-запросов. Это защищает от ситуации, когда чужой сайт пытается отправить действие от имени пользователя.

4. Clickjacking.

   Django может ставить заголовок `X-Frame-Options`. Он мешает открыть сайт внутри чужого `iframe`.

5. Host header attacks.

   Настройка `ALLOWED_HOSTS` говорит Django, каким доменам можно доверять. Это защищает от подмены заголовка `Host`.

6. Пароли и cookies.

   Django хранит пароли как хеши с солью. Еще можно настроить cookies безопаснее: `SESSION_COOKIE_SECURE`, `SESSION_COOKIE_HTTPONLY`, `SESSION_COOKIE_SAMESITE`.

Итого: Django помогает против SQL injection, XSS, CSRF, clickjacking, host header attacks и части проблем с cookies и паролями. Но если отключать защиты или писать небезопасный код руками, то фреймворк не спасет.

### 2. Как работают сессии в Django

Сессии в Django работают довольно просто. Браузеру дают cookie с идентификатором сессии. Обычно cookie называется `sessionid`. По этому ключу Django находит данные сессии на сервере.

Обычная схема:

1. Пользователь заходит на сайт.
2. Django создает или находит его сессию.
3. В браузер отправляется cookie `sessionid`.
4. При следующих запросах браузер отправляет эту cookie обратно.
5. `SessionMiddleware` находит нужную сессию.
6. В коде можно работать с `request.session`.

Пример:

```python
request.session["cart_id"] = cart.id
last_cart = request.session.get("cart_id")
```

`request.session` похож на словарь. Туда можно положить значение и потом получить его в другом запросе.

По умолчанию сессии могут храниться в базе данных. Еще Django умеет хранить их в кэше, файлах или signed cookies.

В обычном варианте в cookie лежит только случайный ключ, а не все данные. Это хорошо, потому что пользователь не может просто открыть cookie и поменять себе роль или id.

Если используются signed cookies, то данные лежат в cookie, но они подписаны через `SECRET_KEY`. Подпись помогает понять, что cookie изменили. Но подпись не шифрует данные, это важно.

Сессии связаны с входом в аккаунт. После логина Django записывает в сессию данные пользователя. Потом `AuthenticationMiddleware` добавляет в запрос `request.user`. Поэтому можно проверить `request.user.is_authenticated`.

При логине Django меняет ключ сессии. Это защита от session fixation. При выходе через `logout()` сессия очищается.

Полезные настройки:

- `SESSION_COOKIE_SECURE`: cookie только по HTTPS;
- `SESSION_COOKIE_HTTPONLY`: JavaScript не может читать cookie;
- `SESSION_COOKIE_SAMESITE`: меньше риска в межсайтовых запросах;
- `SESSION_COOKIE_AGE`: время жизни сессии;
- `SESSION_EXPIRE_AT_BROWSER_CLOSE`: закрывать сессию при закрытии браузера.

## Вывод

В работе были найдены простые ошибки: секрет в коде, пароль как обычное поле, пароль без хеша и странная обработка ошибок. По Django вывод такой: фреймворк дает много готовых защит, но ими надо реально пользоваться. Иначе можно самому все сломать.
