Часть 1

Фрагмент кода 1:

#!/usr/bin/env python3
from flask import Flask

from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')

В коде секретный ключ SECRET_KEY прописан прямо в исходном файле. Он попадает в систему контроля версий, и любой, кто получит доступ к репозиторию, увидит и ключ.
Для устранения можно использовать переменные окружения.

import os
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')


Фрагмент кода 2:  

from flask_wtf import Form  
from wtforms import TextField

class LoginForm(Form):  
username = TextField('username')  
password = TextField('password')

Поле для пароля использует обычный TextField вместо PasswordField. В браузере пароль отображается открытым текстом, а не другими символами. Браузер может сохранить введенный пароль в истории полей ввода.

Часть 2
1. Какие наиболее распространенные атаки предотвращает django?
Обеспечивает встроенную защиту от:
    - SQL-инъекций
    - межсайтового скриптинга (XSS)
    - подделки межсайтовых запросов (CSRF)

2. Описать, как устроено управление пользовательскими сессиями в django
Django использует серверный подход к управлению сессиями: данные хранятся на сервере, а клиент (браузер) получает только идентификатор сессии в виде cookie.

SessionMiddleware управляет сессиями во время запросов. AuthenticationMiddleware ассоциирует пользователей с запросами с помощью сессий. По умолчанию данные сессии хранятся в таблице базы данных django_session.
Некоторые настраиваемые параметры:
    SESSION_COOKIE_AGE: Время жизни сессии в секундах (по умолчанию 2 недели).
    SESSION_COOKIE_DOMAIN: Домен для cookie.
    SESSION_EXPIRE_AT_BROWSER_CLOSE: Если True, сессия удаляется при закрытии браузера.