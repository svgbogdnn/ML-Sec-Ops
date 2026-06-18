# Лабораторная работа 4
## Часть 1
### CVE за 2021 год, содержащие XSS в компонентах, написанных на php
#### 1. CVE-2021-43808 - Laravel
Laravel - очень известный PHP-фреймворк для разработки веб-приложений. В официальной документации прямо сказано, что Laravel - это PHP web application framework. Уязвимость CVE-2021-43808 была связана с компонентом Blade, то есть встроенным шаблонизатором Laravel. Blade нужен для формирования HTML-шаблонов страниц, повторного использования макетов, секций и компонентов представления. Иными словами, через него разработчик собирает интерфейс сайта из шаблонов. В записи NVD указано, что XSS затрагивал именно Blade templating engine.

#### 2. CVE-2021-30157 - MediaWiki
MediaWiki - свободное wiki-приложение, на котором работает Wikipedia и многие другие wiki-сайты. У MediaWiki есть страницы Special:RecentChanges и Special:Watchlist. Они нужны для отслеживания изменений: Recent Changes показывает последние правки, а Watchlist позволяет следить за страницами, которые пользователь добавил в список наблюдения. В описании CVE-2021-30157 сказано, что проблема была именно на ChangesList special pages, таких как Special:RecentChanges и Special:Watchlist, где часть сообщений выводилась в HTML без нужного экранирования, что и приводило к XSS.

#### Общее количество найденных CVE за 2021 год
Помимо двух разобранных примеров, я встретил еще CVE-2021-24202 (Elementor), CVE-2021-42078 (PHP Event Calendar), CVE-2021-43698 (phpWhois), CVE-2021-43683 (pictshare), CVE-2021-27330 (Triconsole Datepicker Calendar), CVE-2021-27673 (Zenario CMS), CVE-2021-26304 (PHPGurukul Daily Expense Tracker System), CVE-2021-40909 (sourcecodester PHP CRUD), CVE-2021-37391 (Chamilo LMS) и CVE-2021-36551 (TikiWiki). Получается я нашел как минимум **12**.

#### Список используемой литературы:
- **NVD. CVE-2021-43808**  
[https://nvd.nist.gov/vuln/detail/CVE-2021-43808](https://nvd.nist.gov/vuln/detail/CVE-2021-43808)
- **Laravel Documentation. Laravel - The PHP Framework for Web Artisans**  
[https://laravel.com/](https://laravel.com/?utm_source=chatgpt.com)
- **Laravel Documentation. Blade Templates**  
[https://laravel.com/docs/blade](https://laravel.com/docs/blade)
- **Debian Security Tracker. CVE-2021-30157**  
[https://security-tracker.debian.org/tracker/CVE-2021-30157](https://security-tracker.debian.org/tracker/CVE-2021-30157?utm_source=chatgpt.com)
- **MediaWiki Documentation. Manual:What is MediaWiki?**  
https://www.mediawiki.org/wiki/Manual:What_is_MediaWiki%3F/en
- **MediaWiki Documentation. Help:Recent changes**  
https://www.mediawiki.org/wiki/Help:Recent_changes
- **MediaWiki Documentation. Manual:Watchlist**  
https://www.mediawiki.org/wiki/Manual:Watchlist

___
## Часть 2
Сперва я запустил докер-контейнер. Перешел в браузер по локалхосту 8080
![[Pasted image 20260418155609.png]]

Далее перешел в приложение XSS-1. Здесь можно было добавить комментарий и сохранить его.
![[Pasted image 20260428210149.png]]

Сначала я протестировал ввел `test` и данный комментарий появился в блоке Comments. То есть приложение приняло ввод, сохранило его и вывело обратно на HTML-страницу. 
![[Pasted image 20260428210237.png]]

Потом я проверил обрабатывает ли приложение текст как обычную строку или как HTML-разметку. Я ввел `<h1>test</h1>` и текст отобразился как крупный жирного шрифта заголовок. То есть приложение не экранирует HTML-теги
![[Pasted image 20260428210258.png]]

Потом я решил проверить, возможно ли выполнить JavaScript-код. Я ввел `<script>alert('XSS')</script>`. После сохранения комментария браузер показал всплывающее окно с текстом XSS. То есть браузер выполнил JavaScript-код, который был введен через поле комментария.
![[Pasted image 20260428210327.png]]

После этого я решил обновить страницу и снова появилось то же всплывающее окно.  То есть JavaScript-код был не просто отражен в ответе на один запрос, а сохранен приложением. И при повторном обновлении страницы снова выводило сохраненный комментарий. Таким образом - атака успешно была проведена. 
![[Pasted image 20260428211701.png]]

### Тип найденной уязвимости - хранимый (stored) XSS
Хранимый XSS — это уязвимость, при которой вредоносный JavaScript-код сохраняется на стороне приложения.

### Меры предотвращения
1. Считать все данные от пользователя недоверенными.  
2. Проверять данные перед сохранением в базу данных.  
3. Не сохранять в комментариях HTML-теги и JavaScript-код, если поле предназначено только для обычного текста.  
4. Использовать белый список разрешенных символов. Например, для комментариев разрешать буквы, цифры, пробелы и обычные знаки препинания.  
5. Экранировать данные при выводе из базы данных на страницу.
6. Проверять и блокировать опасные конструкции:  
	- `<script>`  
	- `</script>`
	- `javascript:`
	- `onload=`
	- `data:`
7. Проводить тестирование формы комментариев на хранимфй XSS.  
8. Преобразовывать специальные символы в HTML-сущности:  
	- `<` → `&lt;`  
	- `>` → `&gt;`  
	- `"` → `&quot;`  
	- `'` → `&#039;`  
	- `&` → `&amp;`
9. Регулярно проверять приложение автоматическими анализаторами и обновлять используемые библиотеки.