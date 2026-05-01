Часть I

Фрагмент 1
```
from flask import Flask
from mod_api import mod_api

app = Flask('vuln_app')
app.config['SECRET_KEY'] = 'F0cUzh8BgYJSLXAU8qDmClM0dE8GJTpsiyVEl3BCqQMCABp1U$f%'

app.register_blueprint(mod_api, url_prefix='/api')
```

Недостаток: SECRET_KEY зашит прямо в исходниках. Этот ключ Flask использует для сессионных кук и CSRF-токенов. В случае утечки, злоумышленник модет:
1)сам подписывать сессионные куки и логиниться под любым юзером
2)подделывать CSRF-токены
3)подделывать любые подписанные данные

Плюс рядом проблема: имя приложения захардкожено как vuln_app, режим отладки нигде не выключен явно - если случайно app.run(debug=True), Werkzeug даст shell прямо в браузере.

как избежать:
1)Ключ читать из переменной окружения, а не из кода: app.config['SECRET_KEY'] = os.environ['SECRET_KEY']. В дев-окружении - из .env который в .gitignore
2)Ключ должен быть длинным и случайным
3)Если ключ когда-то лежал в git - заменить и сделать ротацию, считать его скомпрометированным навсегда
4)Хранить секреты в Vault / AWS Secrets Manager / Kubernetes Secrets, а не в app.config как строку
5)В CI прогонять сканеры секретов (gitleaks, trufflehog), чтобы такое не пушилсь в репозиторий

Фрагмент 2
```
from flask_wtf import Form
from wtforms import TextField

class LoginForm(Form):
    username = TextField('username')
    password = TextField('password')
```

Недостатки:
1)Поле password объявлено как TextField вместо PasswordField. В рендере {{ form.password }} это даст <input type="text"> - пароль виден на экране, его пишет в автозаполнение и сохраняет в браузерной истории форм
2)Ни на одном поле нет валидаторов: можно отправить пустую строку, гигабайт текста, что угодно. Нет DataRequired, Length, Email и т.д
3)В новых версиях wtforms TextField удален, нужно StringField. Это и сигнал что код старый и его никто не поддерживает
4)flask_wtf.Form - устаревший базовый класс, актуальный это FlaskForm. CSRF включен по умолчанию только если используется FlaskForm и сконфигурирован SECRET_KEY

как избежать
1)password = PasswordField('password', validators=[DataRequired(), Length(min=8)])
2)username = StringField('username', validators=[DataRequired(), Length(min=3, max=50)])
3)Наследоваться от FlaskForm, не от Form, чтобы CSRF-защита работала из коробки
4)В шаблоне обязательно рендерить CSRF-токен ({{ form.hidden_tag() }})
5)Дополнительно добавить рейтлимит на эндпоинт логина, чтобы нельзя было подобрать пароли

Фрагмент 3

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

Недостатки
1)Пароль сохраняется в открытом виде. Берется как есть из POST, кодируется в utf-8 и улетает в БД. Никакого bcrypt/argon2/scrypt - при сливе базы все пароли утекут открытым текстом. Это самое тяжелое
2)Голый except Exception который проглатывает все ошибки и показывает «успех». Юзер не создался, валидация упала, БД недоступна - а в ответе success_create.html. Это и UX-провал, и security: атакующий не понимает что пошло не так, но и легитимный юзер думает что зарегался, а его нет. Плюс маскируются реальные ошибки от мониторинга
3)Никакой валидации входа до создания объекта. username, email берутся как есть, email не проверяется на формат, username не проверяется на длину/допустимые символы. Если validate() что и проверяет - все равно его исключение проглотит except
4)password лежит в dict рядом с обычными полями - легко случайно залогировать или вернуть в API.
5)Синтаксис except Exception, e - это Python 2. Код не запустится на Python 3 (нужен except Exception as e). Сигнал что либо код мертвый, либо запускается на EOL-интерпретаторе без обновлений безопасности

как избежать
1)Хешировать пароль перед сохранением В БД писать только хеш, исходный пароль не хранить и не логировать.
2)Заменить except Exception на конкретные классы (ValidationError, IntegrityError) и возвращать осмысленные коды (400 на валидации, 409 на конфликте имени, 500 только на действительно непредвиденных)
3)Валидировать вход явно: длина username, формат email через email-validator, политика паролей (минимум 8 символов, не из топ-1000 утекших)
4)Никогда не показывать success_create.html если транзакция не закоммитилась
5)Перевести код на Python 3 (except ... as e), password обнулять в памяти после хеширования

Вывод
Три фрагмента покрывают три классические дыры: захардкоженный секрет, неправильно собранную форму без CSRF и плейнтекст-пароли с проглоченными исключениями. Для защиты нужно читать секреты из окружения, использовать готовые формы с валидаторами, хешировать пароли через bcrypt/argon2 и не ловить Exception всем подряд

Часть II

Вопрос 1. Какие наиболее распространенные атаки предотвращает Django

Django из коробки закрывает почти весь OWASP Top 10 для веб-приложений:

1)SQL-инъекции. ORM использует параметризованные запросы - User.objects.filter(username=username) не клеит строку, а передает параметр драйверу БД. Сырые запросы (raw(), extra()) тоже принимают параметры отдельно от SQL
2)XSS. Шаблонизатор Django по умолчанию экранирует все переменные при рендере ({{ user_input }} превращает <script> в &lt;script&gt;). Чтобы отключить - надо явно написать |safe или mark_safe, то есть отстрелить ногу можно только осознанно
3)CSRF. Middleware CsrfViewMiddleware включен по умолчанию. Каждая POST-форма должна содержать {% csrf_token %}, иначе запрос отклоняется. Токен привязан к сессии и проверяется по Origin/Referer
4)Clickjacking. Middleware XFrameOptionsMiddleware ставит заголовок X-Frame-Options: DENY - сайт нельзя встроить в <iframe> и заставить пользователя кликнуть «вслепую»
5)Host header injection. Настройка ALLOWED_HOSTS - Django отвергает запросы с чужим Host, что закрывает password reset с подменой домена в письме
6)Open redirect. Утилиты вроде redirect() и url_has_allowed_host_and_scheme проверяют что редирект идет на свой домен, а не на attacker.com
7)Хранение паролей в плейнтексте. User.set_password() кладет в БД хеш PBKDF2 с солью (по умолчанию), либо argon2/bcrypt если они в PASSWORD_HASHERS. Сравнение через check_password()
8)Timing-атаки на сравнение токенов
9)SSL/TLS. Настройки SECURE_SSL_REDIRECT, SESSION_COOKIE_SECURE, CSRF_COOKIE_SECURE, заголовок HSTS (SECURE_HSTS_SECONDS)
10)Загрузка файлов. FileField валидирует размер, имя санитизируется, статика отдается отдельным сервером без интерпретации

Все это работает по умолчанию или включается одной строкой в settings.py. Главное правило - не отключать middleware и не использовать |safe без причины

Вопрос 2. Как устроено управление пользовательскими сессиями в Django

Сессия в Django это словарь, привязанный к пользователю и живущий между запросами. С точки зрения юзера это кука sessionid. С точки зрения сервера - запись в каком-то хранилище плюс middleware которое все это связывает

Что происходит когда юзер открывает сайт:
1)Включенный по умолчанию SessionMiddleware смотрит куку sessionid. Если ее нет - сессия пустая, request.session это пустой dict
2)Если куки есть middleware идет в backend и достает по этому ключу данные сессии, кладет их в request.session
3)View может читать и писать туда что угодно: request.session['cart_id'] = 42
4)На выходе response middleware видит что сессия изменилась (session.modified = True) - сохраняет ее в backend и при необходимости обновляет куку

Где хранятся данные:
Это настройка SESSION_ENGINE, на выбор:
1)django.contrib.sessions.backends.db в БД, таблица django_session с полями session_key, session_data, expire_date
2)cached_db сначала кеш , при промахе БД
3)cache только в кеше, быстро но при рестарте все слетает
4)file  в файлах на диске, для дев-окружения
5)signed_cookies сами данные лежат прямо в куке у клиента, подписаны SECRET_KEY. Никакого хранилища не нужно, но размер ограничен 4 КБ, и любой кто украл куку получает не только id сессии но и ее содержимое

Что лежит в куке:
Только идентификатор сессии (session_key) случайная строка из ~32 символов, сгенеренная secrets.token_hex. Никаких имен/паролей в ней нет, по умолчанию ставится с флагами:
1)HttpOnly (SESSION_COOKIE_HTTPONLY=True) JS не может ее прочитать через document.cookie, защита от кражи через XSS
2)Secure (SESSION_COOKIE_SECURE) включается отдельно, передавать только по HTTPS
3)SameSite=Lax по умолчанию защита от CSRF на cross-site запросах
4)Max-Age / Expires время жизни через SESSION_COOKIE_AGE

Логин и привязка к юзеру:
Когда вызывается django.contrib.auth.login(request, user):
1)Генерируется новая пустая сессия (старый session_key отбрасывается - защита от ession fixation
2)В сессию пишется _auth_user_id, _auth_user_backend, _auth_user_hash
3)При следующих запросах AuthenticationMiddleware читает эти ключи и подставляет request.user

logout(request) очищает данные сессии и удаляет запись из backend

1)Если SESSION_SAVE_EVERY_REQUEST=True срок жизни обновляется на каждом запросе
2)SESSION_EXPIRE_AT_BROWSER_CLOSE=True кука без Max-Age удаляется при закрытии браузера
3)request.session.set_expiry(seconds) поштучно для конкретной сессии

Что есть из коробки против атак на сессию:
1)ротация session_key при логине
2)инвалидация всех сессий юзера при смене пароля
3)HttpOnly + Secure + SameSite куки
4)подпись сессионных данных через SECRET_KEY в signed_cookies это критично, в обычных backend-ах данные также подписаны
Вывод
Django закрывает базовый набор атак на уровне фреймворка - не нужно ничего делать руками, главное не отключать защиту. Сессии работают через middleware + backend: юзеру выдается случайный session_key в защищенной куке, данные хранятся на сервере (БД/кеш), при логине ключ ротируется чтобы исключить session fixation, при смене пароля все сессии юзера автоматически невалидируются
