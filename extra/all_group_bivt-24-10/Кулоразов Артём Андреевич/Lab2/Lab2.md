# Лабораторная работа №2 (REST API)

## Часть 1
### CVE-2025-20349 (Cisco Catalyst Center)
**Источник:** https://nvd.nist.gov/vuln/detail/CVE-2025-20349

**Недостаток:** Аутентифицированный удалённый злоумышленник с ролью Observer может отправить специально созданный запрос к REST API и выполнить произвольные команды в ограниченном контейнере с правами root.

**Причина:** Недостаточная проверка пользовательского ввода в параметрах запроса REST API.

**Категория:** Security Misconfiguration (некорректная валидация ввода, приводящая к инъекции команд).


### CVE-2025-10611 (WSO2 Products)
**Источник:** https://nvd.nist.gov/vuln/detail/CVE-2025-10611

**Недостаток:** Из-за недостаточной реализации контроля доступа в нескольких продуктах WSO2 можно обойти проверки аутентификации и авторизации для определённых REST API. Это позволяет вызывать их без валидации и выполнять административные операции без авторизации.

**Причина:** Недостаточный контроль доступа — отсутствие проверки прав на уровне отдельных функций API.

**Категория:** Broken Function Level Authorization.


### CVE-2025-61757 (Oracle Fusion Middleware Identity Manager)
**Источник:** https://nvd.nist.gov/vuln/detail/CVE-2025-61757

**Недостаток:** В компоненте REST WebServices продукта Oracle Identity Manager отсутствует требование аутентификации для критической функции. Неаутентифицированный злоумышленник с доступом по HTTP может получить полный контроль над Identity Manager.

**Причина:** Отсутствие механизма аутентификации для важного эндпоинта REST API.

**Категория:** Broken Authentication.



## Часть 2

Запустим контейнер:
docker run -d -p 8080:8080 --name rest-lab ket9/otus-devsecops-owasp-rest:latest

Перейдём http://localhost:8080/createdb
Ответ: {"message":"Database populated."}

### Регистрация и вход

Выполним регистрацию пользователя user1:
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/register" -Method Post -Body '{"username":"user1","password":"12345","email":"user1@test.com"}' -ContentType "application/json"
Ответ: {"message":"Successfully registered. Login to receive an auth token.","status":"success"}

Повторная отправка того же запроса выводит: {"status":"fail","message":"User already exists. Please Log in."}
— регистрация работает, дубликаты не создаются.

Выполним вход:
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/login" -Method Post -Body '{"username":"user1","password":"12345"}' -ContentType "application/json"
Ответ: {"auth_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzY4NjQ5MzUsImlhdCI6MTc3Njg2NDg3NSwic3ViIjoidXNlcjEifQ.79AopA2GRwR9u44jvZdra4_wETVsD50t2M6RPSkVMP4"}

Скопируем токен в переменную $token:
$token = $response.auth_token

Проверим свой профиль:
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/user1" -Headers @{Authorization="Bearer $token"}
Ответ: {"username":"user1","email":"user1@test.com"}
Авторизация работает.

### Проверка доступа к чужим данным

Зарегистрируем второго пользователя user2:
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/register" -Method Post -Body '{"username":"user2","password":"12345","email":"user2@test.com"}' -ContentType "application/json"
Ответ: {"message":"Successfully registered. Login to receive an auth token.","status":"success"}

Попробуем посмотреть его профиль с токеном user1:
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/user2" -Headers @{Authorization="Bearer $token"}
Ответ: {"username":"user2","email":"user2@test.com"}

Вывод: сервер не проверяет, имеет ли текущий пользователь право просматривать данные другого пользователя. Это BOLA (Broken Object Level Authorization).

### Попытка удаления

Согласно описанию, удалять пользователей может только админ.

Попробуем удалить user2:
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/user2" -Method Delete -Headers @{Authorization="Bearer $token"}
Ответ: {"status":"fail","message":"Only Admins may delete users!"}

Попробуем удалить самого себя (user1):
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/user1" -Method Delete -Headers @{Authorization="Bearer $token"}
Ответ: {"status":"fail","message":"Only Admins may delete users!"}

Вывод: защита на удаление реализована корректно — обычный пользователь не может вызвать эту функцию даже для своего аккаунта.

### Скрытый эндпоинт _debug

В списке эндпоинтов есть /users/v1/_debug с пометкой «посмотреть подробную информацию обо всех пользователях». Попробуем получить к нему доступ:
Invoke-RestMethod -Uri "http://localhost:8080/users/v1/_debug" -Headers @{Authorization="Bearer $token"} | ConvertTo-Json -Depth 10

Ответ:
{
    "users": [
        { "admin": false, "email": "mail1@mail.com", "password": "pass1", "username": "name1" },
        { "admin": false, "email": "mail2@mail.com", "password": "pass2", "username": "name2" },
        { "admin": true,  "email": "admin@mail.com", "password": "pass1", "username": "admin" },
        { "admin": false, "email": "user1@test.com", "password": "12345", "username": "user1" },
        { "admin": false, "email": "user2@test.com", "password": "12345", "username": "user2" }
    ]
}

Вывод: любой аутентифицированный пользователь может получить пароли всех учётных записей и узнать, кто является администратором. Это критическая уязвимость — Security Misconfiguration и Broken Object Property Level Authorization.

## Итог: найденные уязвимости

1. Просмотр чужих профилей через /users/v1/{username}.
   Тип: Broken Object Level Authorization.

2. Раскрытие паролей и признака admin через эндпоинт /_debug.
   Тип: Broken Object Property Level Authorization + Security Misconfiguration.

## Меры предотвращения

- Для всех эндпоинтов, работающих с конкретным пользователем (/{username}), проверять, что username совпадает с именем владельца токена, либо текущий пользователь имеет роль admin.
- Убрать отладочный эндпоинт /_debug из боевого окружения или ограничить его доступ только администраторам.
- Никогда не хранить и не передавать пароли в открытом виде — использовать хеширование (bcrypt, argon2) и не включать поле password в ответы API.