# Базовая настройка безопасности docker-контейнера

## 1. Запуск базового образа и проверка пользователя

В качестве объекта взял образ `ket9/otus-devsecops-owasp-rest:latest`. Это VAMPI, vulnerable API for testing на Flask, упакованный в Alpine Linux. Перед запуском убедился, что на хосте я под обычным пользователем:
```
$ whoami
fatuum
```

Список локальных образов:
```
$ docker image ls
IMAGE                                    ID             DISK USAGE
ket9/otus-devsecops-owasp-rest:latest    080abc666726   312MB
ket9/otus-devsecops:latest               5585d312a54b   472MB
mysql:5.7                                5107333e08a8   501MB
alpine:latest                            2ffb2ff4aab3   8.7MB
```

Дальше зашёл внутрь контейнера. Сразу словил прикол: у образа задан `ENTRYPOINT ["python"]`, поэтому привычное `docker run -it ... sh` интерпретируется как `python sh` и Python пытается открыть файл с именем "sh". Пришлось переопределить точку входа:
```
$ docker run -it --entrypoint sh ket9/otus-devsecops-owasp-rest:latest
/vampi # whoami
root
/vampi # id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

Приложение работает из под root с полным набором базовых групп. Если в API найдётся RCE (а у VAMPI он там есть by design), атакующий сразу получит root внутри контейнера. При небезопасной конфигурации контейнера это может быть не только проблема внутри контейнера, но и риск для хоста

- тут должен быть скриншот 1(см. в файлах гугл диска)


## 2. Dockerfile с непривилегированным пользователем

Написал Dockerfile поверх того же базового образа, но с переключением на отдельного системного пользователя:
```dockerfile
FROM ket9/otus-devsecops-owasp-rest:latest

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /vampi
RUN chown -R appuser:appgroup /vampi

USER appuser

EXPOSE 5000
```

Использовал alpine-синтаксис `addgroup -S` / `adduser -S`, потому что базовый образ собран на alpine 3.14. Для debian-based образа потребовался бы `groupadd -r` / `useradd -r`

Сборка:
```
$ docker build -t my-secure-rest .
[+] Building 4.3s (8/8) FINISHED
 => [stage-1 1/4] FROM docker.io/ket9/otus-devsecops-owasp-rest:latest
 => [stage-1 2/4] RUN addgroup -S appgroup && adduser -S appuser -G appgroup
 => [stage-1 3/4] WORKDIR /vampi
 => [stage-1 4/4] RUN chown -R appuser:appgroup /vampi
 => => writing image sha256:b38e11fd0896...
```

Запуск и проверка:
```
$ docker run -it --rm --entrypoint sh my-secure-rest
/vampi $ whoami
appuser
/vampi $ id
uid=101(appuser) gid=101(appgroup) groups=101(appgroup)
```

Цель достигнута: в моём образе приложение стартует от `appuser` с uid 101.

- тут должен быть скриншот 2(см. в файлах гугл диска)

USER appuser снижает риск при взломе приложения, но сам по себе не закрывает CVE в пакетах и не убирает лишние привилегии контейнера. Поэтому в комментариях к Dockerfile отдельно перечислены флаги `docker run`, которые имеет смысл выставлять вместе. Самый болезненный антипаттерн, который видел в проде: проброс `/var/run/docker.sock` внутрь контейнера, чтобы CI-раннер мог собирать образы. Это эквивалент рутового доступа на хост, и никакой `USER appuser` тут не поможет.


## 3. Сканирование образа trivy

Поставил trivy, версия 0.70.0. Прогнал по обоим образам:
```
$ trivy image --severity HIGH,CRITICAL ket9/otus-devsecops-owasp-rest:latest
$ trivy image --severity HIGH,CRITICAL my-secure-rest
```

Саммари по оригинальному образу:
```
ket9/otus-devsecops-owasp-rest:latest (alpine 3.14.2)
Total: 56 (HIGH: 48, CRITICAL: 8)

Python (python-pkg)
Total: 14 (HIGH: 14, CRITICAL: 0)
```

Самое важное trivy сказал ещё до таблицы:
```
WARN This OS version is no longer supported by the distribution
     family="alpine" version="3.14.2"
WARN The vulnerability detection may be insufficient because security
     updates are not provided
```

Alpine 3.14 снят с поддержки. Новых патчей под него не выпускают, и сколько ни ставь `apk upgrade`, фиксов всё равно нет. Это и есть главный системный вывод по образу: лечить нужно не отдельные CVE, а сам базовый слой.

Из конкретики выбрал три находки, которые показались наиболее болезненными.

**zlib CVE-2022-37434, CRITICAL.** 
Уязвимость связана с чтением данных за пределами буфера в функции inflate() из файла inflate.c. В образе установлена версия 1.2.11-r3, исправление есть начиная с 1.2.12-r2. Проблема может проявиться при обработке gzip потока с нестандартным дополнительным полем в заголовке. Для веб-приложения это опасно, если оно или его библиотеки разбирают сжатые данные из недоверенного источника. В таком случае специально подготовленный gzip запрос или ответ от внешнего сервиса может привести к сбою процесса. Так как zlib используется во многих местах, например в Python, OpenSSL и HTTP-сжатии, такая уязвимость затрагивает не один отдельный компонент, а сразу несколько слоев приложения

**openssl CVE-2022-0778, HIGH.** 
Бесконечный цикл в `BN_mod_sqrt()` при парсинге сертификатов с невалидными параметрами эллиптической кривой. Установлено `1.1.1l-r0`, фикс в `1.1.1n-r0`. Чисто DoS, но запускается одним битым сертификатом. У VAMPI mTLS, скорее всего, не используется, но libcrypto всё равно дёргается через requests и certifi, поэтому уязвимость отстреливает при походе в любую недоверенную TLS-точку

**Flask CVE-2023-30861, HIGH.**
 Установлен `Flask 1.1.4`, фикс в `2.2.5` или `2.3.2`. Flask не выставляет заголовок `Vary: Cookie` в ответах с сессионной кукой при определённых сочетаниях `session_protection` и кешируемых эндпоинтов. Если перед приложением стоит CDN или nginx с `proxy_cache`, кеш может отдать сессионную куку одного пользователя другому. На лабораторной инсталляции это не выстрелит, в проде это утечка сессий


Прогнал свой собранный образ:
```
my-secure-rest (alpine 3.14.2)
Total: 56 (HIGH: 48, CRITICAL: 8)
```

Цифры идентичные. Это ожидаемо: я добавил только пользователя и `WORKDIR`, никаких пакетов не обновлял. Полезный момент для понимания: переключение на не-root пользователя не лечит CVE в библиотеках, оно только снижает blast radius при эксплуатации.

- тут должен быть скриншот 3(см. в файлах гугл диска)


### Что я бы делал с этим образом дальше

Главная проблема не в конкретных CVE, а в том, что база EOL. Пока её не обновят, любой свежий аудит будет давать ту же гору HIGH/CRITICAL. Порядок приоритетов:

1. Перейти на актуальный базовый образ (alpine 3.20 или официальный python:3.12-alpine). Это уберёт большую часть системных CVE одним движением.
2. Обновить Python-зависимости. Особенно Flask, Werkzeug, PyJWT, certifi, потому что они на пути пользовательского трафика.
3. Только после этого думать про runtime-настройки. `USER appuser`, `--read-only`, `--cap-drop=ALL`, `no-new-privileges`, `--pids-limit` сами по себе CVE не закрывают, но делают пост-эксплуатацию заметно дороже: даже если атакующий найдёт RCE через дырявый zlib или Werkzeug, он попадёт в read-only контейнер под appuser без capabilities, откуда выход на хост это уже отдельная история.
