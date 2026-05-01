### Часть 1

1. 
```
#!/usr/bin/env python3
from flask import Flask

from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')

```
Здесь SECRET_KEY прописан явно. Если такой код запушить в репозиторий, то все увидят ключ, что может привести к тому, что кто угодно может подделывать сессии и расшифровывать данные.
Чтобы устранить этот недостаток, можно использовать переменные окружения 
```
import os
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
``` 

2. 
   ```
from flask_wtf import Form  
from wtforms import TextField

class LoginForm(Form):  
username = TextField('username')  
password = TextField('password')
   ```
Поле пароля определено, как текстовое. Основные причины, почему так не стоит делать: при вводе пароль будет отображаться текстом, а не какими-то символами (точки или звёздочки), пароль может сохраниться в кеше. 
Устранить недостаток можно, если использовать формат поля PasswordField: 
```
from flask_wtf import Form  
from wtforms import TextField

class LoginForm(Form):  
username = TextField('username')  
password = PasswordField('password')
```

3. 
```
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
Пароль записывается в явном виде, никак не шифруется. Нет проверки почты на формат. При ошибке в процессе регистрации пользователь попадает на страницу success_create.html, хотя так не должно быть, ведь случилась ошибка, значит, create не success. 
Решения проблем:
- хешировать или как-то шифровать пароли
- проверять формат почты 
- при ошибке регистрации выдавать какую-то не успешную страницу (а success_create.html можно показывать при успешной регистрации)

---

### Часть 2
1. Django: 
   - защищает от XSS атак
   - защищает от SQL- инъекций
   - обрабатывает пароли (хеширует)
   - использует HTTPS
2.  Управление пользовательскими сессиями в Django: 
   - два компонента: SessionMiddleware - управление запросом между сессиями; AuthenticationMiddleware - связывает пользователя и сессию 
   - данные сессии храняться в БД (по умолчанию), кэше, Cookies и файловой системе 
   - Django позволяет настраивать различные параметры поведения сессий, например:
	- SESSION_COOKIE_AGE — устанавливает время истечения сессии в секундах.
	- SESSION_COOKIE_SECURE — требует передачи сессий по HTTPS.
	- SESSION_EXPIRE_AT_BROWSER_CLOSE — завершает сессию при закрытии браузера.
- Безопасность: 
  - безопасная обработка куков
  - требование использования HTTPS
  - настройка истечения сессий