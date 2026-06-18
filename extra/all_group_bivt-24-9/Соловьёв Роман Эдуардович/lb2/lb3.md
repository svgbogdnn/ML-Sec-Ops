##Часть I
###CVE, относящиеся к недостаткам REST API из OWASP API Top 10

**CVE-2023-45158**
Уязвимость обнаружена в REST API плагина для WordPress. Любой аутентифицированный пользователь мог изменять данные других пользователей, просто подставив чужой числовой id в URL-запросе — никакой проверки прав на уровне объекта не было. Причина в том, что разработчики проверяли только факт наличия токена, но не сверяли, принадлежит ли запрашиваемый объект текущему пользователю. Относится к API1:2023 — Broken Object Level Authorization.
**CVE-2024-21683**
Уязвимость в REST API Confluence от Atlassian. Ряд административных эндпоинтов не требовал никакой аутентификации — удалённый атакующий мог отправить запрос напрямую и выполнить произвольный код на сервере. Причина в том, что функциональные эндпоинты оказались не прикрыты middleware авторизации. Относится к API5:2023 — Broken Function Level Authorization.
**CVE-2024-27564**
Уязвимость в REST API сервиса на базе ChatGPT. Через параметр imageUrl можно было передать произвольный внутренний адрес — сервер послушно отправлял туда запрос, фактически открывая SSRF. Причина в отсутствии валидации и фильтрации входных URL. Относится к API7:2023 — Server Side Request Forgery.

###Ссылки

1. https://nvd.nist.gov/vuln/detail/CVE-2023-45158
2. https://nvd.nist.gov/vuln/detail/CVE-2024-21683
3. https://nvd.nist.gov/vuln/detail/CVE-2024-27564

##Часть II
Поднял образ командой docker pull ket9/otus-devsecops-owasp-rest, затем docker run -d -p 8080:8080 ket9/otus-devsecops-owasp-rest:latest. Зашёл на http://localhost:8080 — приложение отдало список доступных эндпоинтов. Наполнил базу http://localhost:8080/createdb.

Начал с разведки: запросил http://localhost:8080/users/v1 — сервер вернул список пользователей с базовыми полями, без паролей. Среди эндпоинтов увидел /users/v1/\_debug с описанием подробная информация обо всех пользоваттелях. Слово debug насторожило.

Запросил http://localhost:8080/users/v1/\_debug без авторизации. Сервер вернул полный список пользователей включая поля password: name1/pass1, name2/pass2, admin/pass1. Я выяснил, что это первая уязвимость — дебаг-эндпоинт доступен в production без авторизации.

Дальше решил проверить, можно ли используя свой аккаунт изменить данные чужого. Зарегистрировался через POST http://localhost:8080/users/v1/register с телом {"username": "testuser", "password": "test123", "email": "test@test.com"}, затем залогинился и одной командой сразу получил токен и отправил запрос на смену пароля пользователя name1:
$token = (Invoke-WebRequest -UseBasicParsing -Uri "http://localhost:8080/users/v1/login" -Method POST -ContentType "application/json" -Body '{"username": "testuser", "password": "test123"}' | ConvertFrom-Json).auth_token; Invoke-WebRequest -UseBasicParsing -Uri "http://localhost:8080/users/v1/name1/password" -Method PUT -ContentType "application/json" -Headers @{"Authorization"="Bearer $token"} -Body '{"password": "hacked123"}' | Select-Object -ExpandProperty Content

Команда выполнилась без ошибок. Попробовал залогиниться под name1 со старым паролем:
Invoke-WebRequest -UseBasicParsing -Uri "http://localhost:8080/users/v1/login" -Method POST -ContentType "application/json" -Body '{"username": "name1", "password": "pass1"}' | Select-Object -ExpandProperty Content
Сервер вернул: { "status": "fail", "message": "Password is not correct for the given username."} — старый пароль больше не работает. Затем попробовал с новым:
Invoke-WebRequest -UseBasicParsing -Uri "http://localhost:8080/users/v1/login" -Method POST -ContentType "application/json" -Body '{"username": "name1", "password": "hacked123"}' | Select-Object -ExpandProperty Content
Сервер вернул: {"auth_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....", "message": "Successfully logged in.", "status": "success"} — логин прошёл успешно. Я выяснил, что сервер проверяет только факт наличия токена, но не сверяет, имеет ли текущий пользователь право менять данные запрошенного объекта.

Найденные уязвимости:

1. API9:2023 — Improper Inventory Management. Дебаг-эндпоинт /users/v1/\_debug оказался открыт в production без авторизации и вернул пароли всех пользователей в открытом виде. Причина — разработчики не удалили или не закрыли служебный эндпоинт перед деплоем
2. API1:2023 — Broken Object Level Authorization (BOLA). Аутентифицированный пользователь testuser смог сменить пароль другого пользователя name1, просто указав чужой username в URL. Причина — сервер не проверяет, совпадает ли username в запросе с тем, кому выдан токен

Меры предотвращения:
По API9 — отключать все debug и admin эндпоинты перед деплоем в production, проводить инвентаризацию API и следить за тем, чтобы внутренние эндпоинты не были доступны снаружи.
По API1 — на каждый запрос, затрагивающий объект конкретного пользователя, проверять что sub в токене совпадает с username в пути запроса. Реализовывать object-level authorization в middleware, а не рассчитывать на то, что клиент передаст правильные данные.
