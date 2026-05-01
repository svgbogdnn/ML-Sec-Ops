### Часть 1
- CVE-2026-2025
  API1:2025 - Нарушенный контроль доступа 
  Один из эндпоинтов API был не защищён, поэтому можно было легко достать email адреса в WordPress-плагине Mail Mint, отправив запрос.
  Причина: в коде не было проверки, кем является пользователь (просто пользователь или администратор), поэтому любому желающему выдавались данные 
- CVE-2026-3262
  API3:2025 — Чрезмерное раскрытие данных (Excessive Data Exposure)
  Уязвимость связана с неизвестной функцией компонента Administrative Interface. Манипуляции с ней приводят к выполнению после перенаправления. Это означает, что веб-приложение отправляет перенаправление на другой URL, но вместо завершения работы выполняет дополнительный код.
- CVE-2026-3263
  API1:2021
  Уязвимость в системе управления инвентарём и заказами go2ismail Asp.Net-Core-Inventory-Order-Management-System. Уязвимость обнаружена в версиях системы до 9.20250118. Она затрагивает компонент Security API и функциональность файла /api/Security/, к которому можно было легко отправить запрос и получить доступ к функциям безопасности системы.

---
### Часть 2
1. После запуска образа я вижу это: 
   ```
   { "создать базу": "/createdb",     "посмотреть информацию обо всех пользователях": "/users/v1",     "посмотреть подробную информацию обо всех пользователях": "/users/v1/_debug",     "регистрация нового пользователя": "/users/v1/register (HTTP POST)",     "вход в приложение": "/users/v1/login (HTTP POST)",     "просмотр пользователя по имени": "/users/v1/{username}",     "удалить пользователя по имени (только для админов)": "/users/v1/{username} (HTTP DELETE)",     "изменить email пользователя": "/users/v1/{username}/email (HTTP PUT)",     "изменить пароль пользователя": "/users/v1/{username}/password (HTTP PUT)",     "все книги": "/books/v1",     "добавить книгу": "/books/v1 (HTTP POST)",     "просмотр книги": "/books/v1/{book}",      }
   ```
   То есть я вижу документацию по API
2. Посмотрим тех пользователей, кто уже есть по ссылке http://localhost:8080/users/v1

{"users":[{"email":"mail1@mail.com","username":"name1"},{"email":"mail2@mail.com","username":"name2"},{"email":"admin@mail.com","username":"admin"}]}

И теперь попробуем создать нового пользователя. 
3. В описании API написано, что через /users/v1/register происходит регистрация пользователей. Если ввести это в командной строке, то получу  "status": "fail", "message": "User not found"}, это что-то не то, что я бы хотела увидеть. Поговорила с нейронкой, она мне помогла написать вот такой html файлик
	   ```
	   <form method="POST" action="http://localhost:8080/users/v1/register">

    <input name="username" placeholder="username">

    <input name="password" placeholder="password">

    <input name="email" placeholder="email">

    <button type="submit">Register</button>

</form>
	   ```
	   Запускаю этот файл в браузере и получаю окошечки для регистрации пользователя. 
	   Создаю пользователя poli, пароль 123, почта poli@mil.ru. Получила ошибку 
	   {
  "detail": null,
  "status": 415,
  "title": "Invalid Content-type (application/x-www-form-urlencoded), expected JSON data",
  "type": "about:blank"
}

Окей, тут ошибка в том, что данные подаются в неверном формате, тогда создам пользователя через Power Shell:
```
$body = @{
>>     username = "poli"
>>     password = "123"
>>     email = "poli@mil.ru"
>> } | ConvertTo-Json
```

И ввожу: 
```
 Invoke-RestMethod -Uri "http://localhost:8080/users/v1/register" `
>>     -Method Post `
>>     -Body $body `
>>     -ContentType "application/json"
```
Смотрю пользователей по ссылке http://localhost:8080/users/v1, и вижу 
{"users":[{"email":"mail1@mail.com","username":"name1"},{"email":"mail2@mail.com","username":"name2"},{"email":"admin@mail.com","username":"admin"},{"email":"poli@mil.ru","username":"poli"}]}, то есть мой новый пользователь создался, ура. 
4. Теперь я хочу посмотреть подробную информацию обо всех пользователях по ссылке http://localhost:8080/users/v1/_debug

тут я вижу 
```
{"users":[{"admin":false,"email":"mail1@mail.com","password":"pass1","username":"name1"},{"admin":false,"email":"mail2@mail.com","password":"pass2","username":"name2"},{"admin":true,"email":"admin@mail.com","password":"pass1","username":"admin"},{"admin":false,"email":"poli@mil.ru","password":"123","username":"poli"}]}
```
Здесь мы видим все логины и пароли пользователей. Это уязвимости A01:2021 (нарушение контроля доступа) и A02:2021 (криптографические сбои, так как система не шифрует пароли). 
5. Попробую просто посмотреть информацию о конкретном пользователе по ссылке http://localhost:8080/users/v1/poli
   ```
   {"username": "poli", "email": "poli@mil.ru"}
   ```
   Здесь видно только логин и почту, паролей нет, тут всё окей
6. Этот недостаток относится к двум уязвимостям: 
   - A01:2021 - Broken Access Control
   - A02:2021 - Cryptographic Failures
   Меры предотвращения: 
   - закрыть доступ к эндпоинту и открывать его только для авторизованных администраторов 
   - Не хранить пароли в явном виде, а записывать, например, хэши 
