# лабораторная работа 4

халибеков алан бивт 24 9

## часть 1

### фрагмент 1

```python
#!/usr/bin/env python3
from flask import Flask
from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')
```

недостаток - секретный ключ хранится прямо в коде

почему опасно - если код попадет в репозиторий или в чужие руки то злоумышленник получит ключ и сможет подделывать сессии и токены

как исправить

1. не хранить secret_key в коде
2. загружать из переменных окружения или закрытого конфига
3. использовать разные ключи для разработки и продакшена

### фрагмент 2

```python
from flask_wtf import Form
from wtforms import TextField

class LoginForm(Form):
    username = TextField('username')
    password = TextField('password')
```

недостаток - поле пароля объявлено как обычное текстовое поле textfield

почему опасно - пароль виден на экране в открытом виде нет валидации обязательности и длины

как исправить

1. заменить textfield на passwordfield для пароля
2. добавить валидаторы datarequired и length
3. включить csrf защиту формы

### фрагмент 3

```python
def post(self):
    username = self.get_argument('username')
    password = self.get_argument('password').encode('utf-8')
    email = self.get_argument('mail')
    try:
        username = username.lower()
        email = email.strip().lower()
        user = User({'username': username, 'password': password, 'email': email})
        user.validate()
        save_user(self.db_conn, user)
    except Exception, e:
        return self.render_template("success_create.html")
```

недостатки

1. except exception перехватывает все подряд это плохая практика
2. при ошибке возвращается страница успеха что вводит пользователя в заблуждение
3. пароль сохраняется без хеширования
4. ошибки нигде не логируются

почему опасно - при утечке базы все пароли будут видны в открытом виде реальные ошибки скрываются и это мешает отладке и мониторингу

как исправить

1. ловить только конкретные исключения а не все подряд
2. при ошибке показывать страницу ошибки а не успеха
3. хешировать пароль через bcrypt argon2 или pbkdf2 перед сохранением
4. добавить логирование исключений

## часть 2

### какие атаки предотвращает django

- csrf - django проверяет csrf токен для небезопасных запросов post put delete
- xss - шаблонизатор по умолчанию экранирует спецсимволы в выводе
- sql injection - orm параметризует запросы автоматически пользовательский ввод не попадает в sql напрямую
- clickjacking - поддерживается заголовок x-frame-options который запрещает встраивание страниц в чужие фреймы
- небезопасное хранение паролей - django хеширует пароли встроенными средствами

### управление сессиями в django

сессии позволяют хранить данные о пользователе между запросами

как это работает

1. за сессии отвечает sessionmiddleware
2. при первом запросе django создает сессию и кладет ее идентификатор в cookie браузера
3. при следующем запросе браузер возвращает cookie django находит нужную сессию
4. данные читаются и пишутся через request.session

где хранятся сессии - в базе данных кеше файлах или в подписанных cookie чаще всего используется база с таблицей django_session

что можно хранить - id пользователя факт авторизации временные данные пользовательские настройки

настройки безопасности

- session_cookie_secure - только по https
- session_cookie_httponly - запрет доступа из js
- session_cookie_samesite - защита от межсайтовых атак
- session_cookie_age - время жизни сессии
- session_expire_at_browser_close - сессия завершается при закрытии браузера

при выходе пользователя данные сессии очищаются устаревшие сессии периодически удаляются

## вывод

в первой части разобрали три фрагмента кода с типичными проблемами - ключ в коде пароль в открытом текстовом поле и сохранение пароля без хеша во второй части посмотрели как django защищает от основных атак и как устроены сессии

## источники

1. https://docs.djangoproject.com/en/stable/topics/security/
2. https://docs.djangoproject.com/en/stable/topics/http/sessions/
