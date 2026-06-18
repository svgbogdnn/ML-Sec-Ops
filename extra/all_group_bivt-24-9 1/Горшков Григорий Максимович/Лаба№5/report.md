# Отчет по домашнему заданию

## Цель работы

Научиться находить распространенные недостатки в Python-коде и познакомиться с механизмами безопасности в Django.

## Часть I. Анализ фрагментов кода

### Фрагмент 1

```python
#!/usr/bin/env python3
from flask import Flask

from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')
```

Недостаток: секретный ключ приложения жестко записан в исходном коде.

Почему это опасно: `SECRET_KEY` используется фреймворком для криптографических операций, например для подписи cookies, сессий и CSRF-токенов. Если исходный код попадет в репозиторий, логи, архив или к третьим лицам, ключ считается скомпрометированным. Злоумышленник может попытаться подделывать подписанные данные, а также атаковать механизмы сессий и защиты форм.

Рекомендации:

- хранить секреты вне исходного кода: в переменных окружения, `.env`-файлах без добавления в Git, Vault, Secret Manager или другом хранилище секретов;
- использовать разные ключи для разработки, тестирования и production;
- при утечке ключа немедленно заменить его и инвалидировать старые сессии;
- генерировать ключ криптографически стойким способом.

Пример более безопасного варианта:

```python
import os
from flask import Flask

app = Flask(__name__)
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

Недостаток: форма логина не содержит серверной валидации, а поле пароля объявлено как обычное текстовое поле.

Почему это опасно: без валидаторов приложение может принимать пустые, слишком длинные или некорректные значения. Это повышает риск ошибок, обхода бизнес-логики и проблем при дальнейшей обработке данных. Кроме того, обычное текстовое поле для пароля может отображать вводимый пароль открытым текстом в браузере.

Рекомендации:

- использовать `PasswordField` для пароля;
- добавить валидаторы, например обязательность поля и ограничения длины;
- проверять данные на стороне сервера, а не полагаться только на HTML-атрибуты браузера;
- использовать актуальные классы Flask-WTF и WTForms, например `FlaskForm`, `StringField`, `PasswordField`;
- убедиться, что CSRF-защита Flask-WTF включена и настроен надежный `SECRET_KEY`.

Пример более безопасного варианта:

```python
from flask_wtf import FlaskForm
from wtforms import PasswordField, StringField
from wtforms.validators import DataRequired, Length


class LoginForm(FlaskForm):
    username = StringField(
        "username",
        validators=[DataRequired(), Length(min=3, max=64)],
    )
    password = PasswordField(
        "password",
        validators=[DataRequired(), Length(min=8, max=128)],
    )
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

Недостатки:

- пароль только кодируется в UTF-8, но не хешируется;
- все исключения перехватываются через общий `except Exception`;
- при любой ошибке пользователю показывается страница успешного создания аккаунта.

Почему это опасно: кодирование строки не является защитой пароля. Если пароль сохраняется в таком виде, то при утечке базы данных будут раскрыты реальные пароли пользователей. Общий перехват исключений скрывает ошибки валидации, ошибки базы данных и программные ошибки. Возврат страницы успеха при исключении приводит к неверному поведению приложения и затрудняет расследование инцидентов.

Рекомендации:

- хранить только хеш пароля, созданный специальным алгоритмом для паролей: Argon2, bcrypt, scrypt или PBKDF2;
- использовать готовые функции фреймворка, например `generate_password_hash()` в Werkzeug или встроенные хешеры Django;
- обрабатывать ожидаемые ошибки отдельно: ошибки валидации, конфликт имени пользователя, ошибки базы данных;
- логировать неожиданные исключения и возвращать пользователю корректную страницу ошибки, а не страницу успеха;
- не использовать устаревший синтаксис `except Exception, e`, в Python 3 нужно писать `except Exception as e`.

Пример направления исправления:

```python
from werkzeug.security import generate_password_hash


def post(self):
    username = self.get_argument("username").strip().lower()
    password = self.get_argument("password")
    email = self.get_argument("mail").strip().lower()

    try:
        user = User({
            "username": username,
            "password": generate_password_hash(password),
            "email": email,
            "date_joined": curtime(),
        })
        user.validate()
        save_user(self.db_conn, user)
    except ValidationError:
        return self.render_template("create_user.html", error="Некорректные данные")
    except Exception:
        logger.exception("Unexpected user creation error")
        return self.render_template("create_user.html", error="Не удалось создать пользователя")

    return self.render_template("success_create.html")
```

## Часть II. Модель безопасности Django

### 1. Какие распространенные атаки предотвращает Django

Django содержит встроенные механизмы, которые помогают предотвращать следующие распространенные атаки:

- XSS: шаблонизатор Django по умолчанию экранирует опасные HTML-символы. Это снижает риск внедрения JavaScript в страницы. При этом разработчик должен осторожно использовать `safe`, `mark_safe`, отключение autoescape и вывод данных в HTML-атрибуты.
- CSRF: `CsrfViewMiddleware` и тег `{% csrf_token %}` защищают POST-запросы от выполнения действий от имени пользователя без его согласия. При HTTPS Django также проверяет источник запроса.
- SQL injection: ORM Django строит запросы с параметризацией, отделяя SQL-код от пользовательских параметров. Риск снова появляется при небезопасном использовании raw SQL, `extra()` или `RawSQL`.
- Clickjacking: Django может отправлять заголовок `X-Frame-Options`, который запрещает или ограничивает открытие сайта во фрейме на чужом сайте.
- Host header attacks: Django проверяет заголовок `Host` через настройку `ALLOWED_HOSTS`, если используется стандартный механизм `request.get_host()`.
- Перехват cookies и сессий: настройки `SESSION_COOKIE_SECURE`, `SESSION_COOKIE_HTTPONLY`, `SESSION_COOKIE_SAMESITE`, `CSRF_COOKIE_SECURE`, HTTPS и HSTS уменьшают риск кражи и неправильной отправки cookies.
- Content injection и часть XSS-рисков: в актуальных версиях Django можно использовать middleware для Content Security Policy, который ограничивает допустимые источники скриптов, стилей, изображений и других ресурсов.

Важно: Django не делает приложение безопасным автоматически. Защита может быть ослаблена, если разработчик отключает middleware, использует небезопасные raw-запросы, хранит секреты в коде, неправильно обрабатывает пользовательские файлы или выводит непроверенный HTML.

### 2. Как устроено управление пользовательскими сессиями в Django

Сессии в Django реализованы через приложение `django.contrib.sessions` и middleware `django.contrib.sessions.middleware.SessionMiddleware`. Когда middleware включен, у каждого объекта запроса появляется `request.session`. Это словареподобный объект, через который приложение может читать и записывать данные сессии.

По умолчанию Django хранит данные сессий на стороне сервера в базе данных, в таблице `django_session`. В браузер пользователя отправляется cookie с идентификатором сессии, обычно `sessionid`. В обычном серверном backend cookie содержит не сами данные сессии, а только ключ, по которому Django находит данные на сервере. Исключение: cookie-based backend, где данные сессии лежат в cookie, но они только подписаны, а не зашифрованы, поэтому пользователь может их прочитать.

Основные варианты хранения сессий:

- база данных: backend по умолчанию;
- cache: быстрее, но нужно использовать надежный backend вроде Redis или Memcached;
- cached_db: запись идет и в базу, и в cache;
- файловое хранилище;
- signed cookies: данные хранятся у клиента в подписанной cookie.

Сессия сохраняется не при каждом запросе, а когда она была изменена, например при присваивании или удалении значения в `request.session`. Это поведение можно изменить настройкой `SESSION_SAVE_EVERY_REQUEST`. Cookie отправляется клиенту, когда сессия создана или изменена.

Встроенная система аутентификации Django использует сессии для связи пользователя с запросом. После успешного входа функция `login()` сохраняет ID пользователя и использованный backend аутентификации в сессии. Затем `AuthenticationMiddleware` на следующих запросах восстанавливает пользователя и предоставляет его как `request.user`. Если пользователь не вошел в систему, `request.user` будет `AnonymousUser`.

При выходе из аккаунта функция `logout()` полностью очищает данные текущей сессии. Это нужно, чтобы другой человек, использующий тот же браузер, не получил доступ к данным предыдущего пользователя. При смене пароля Django проверяет session auth hash, связанный с паролем пользователя, и может инвалидировать старые сессии.

За безопасность сессионных cookies отвечают настройки:

- `SESSION_COOKIE_HTTPONLY`: запрещает JavaScript читать session cookie;
- `SESSION_COOKIE_SECURE`: отправляет cookie только по HTTPS;
- `SESSION_COOKIE_SAMESITE`: ограничивает отправку cookie в cross-site-запросах;
- `SESSION_COOKIE_AGE`: задает срок жизни cookie;
- `SESSION_EXPIRE_AT_BROWSER_CLOSE`: позволяет завершать сессию при закрытии браузера;
- `SESSION_COOKIE_DOMAIN`: задает домен cookie, но неправильная настройка может привести к session fixation через недоверенные поддомены.

Также нужно регулярно очищать устаревшие серверные сессии командой `clearsessions`, если используется backend на базе данных или файлов.

## Вывод

В первой части были найдены типичные проблемы: хранение секретного ключа в коде, отсутствие корректной валидации формы, небезопасная обработка пароля и ошибочный перехват исключений. Во второй части рассмотрены ключевые механизмы безопасности Django: защита от XSS, CSRF, SQL-инъекций, clickjacking, атак на Host-заголовок и сессионных атак. Сессии Django построены вокруг cookie с идентификатором сессии и серверного хранилища данных, а безопасность зависит от правильного middleware, настроек cookies, HTTPS и корректного использования встроенной аутентификации.

## Источники

- Django documentation: Security in Django: https://docs.djangoproject.com/en/6.0/topics/security/
- Django documentation: How to use sessions: https://docs.djangoproject.com/en/6.0/topics/http/sessions/
- Django documentation: Using the Django authentication system: https://docs.djangoproject.com/en/6.0/topics/auth/default/
