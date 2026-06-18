# Часть 1

### Фрагмент кода 1:

```#!/usr/bin/env python3
from flask import Flask
from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')
```

Проблема: secret key захардкожен. Если будет найден исходный код, то будет слит и ключ.

### Фрагмент кода 2:

``` #!/usr/bin/env python3
from flask_wtf import Form  
from wtforms import TextField

class LoginForm(Form):  
username = TextField('username')  
password = TextField('password')
```

Проблемы:

- нет проверки текстовых данных
- пароль хранится в открытом виде

### Фрагмент кода 3

```#!/usr/bin/env python3
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

Проблемы: 

- Текстовые данные не валидируется

- В случае Exception всё равно возвращается сообщение об успешном создании пользователя

- Пароль по факту хранится в открытом виде (encoding =/= хэширование)

# Часть 2

Django защищает от следующих уязвимостей:

- XSS (используется url encoding)
- CSRF (используется токен пользователя)
- SQLi (ORM + prepared statements)





В Django используются user sessions. Когда пользователь подключается к серверу в первый раз происходит следующее:

1) Django создает случайный session_id 
2) Отправляет его пользователю
3) Пользователь хранит только свой session_id
4) Django хранит всю информации о сессии и каждый раз обращается в БД по session_id

Почему это безопасно:

- session_id - случайное значение
- кука HttpOnly, то есть она не доступна для JS
- работает по https
- сессии хранятся только определенное время, а потом стираются 