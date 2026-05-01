Часть I

На хосте до запуска контейнера выполнил whoami - вернулся мой логин (Kirill). Дальше посмотрел список локальных образов
```
docker image ls
```
Из имеющихся выбрал ket9/otus-devsecops-owasp-rest:latest. Запустил оболочку внутри:
```
docker run -it --entrypoint sh ket9/otus-devsecops-owasp-rest:latest
```
Внутри контейнера снова whoami - вернулся root. То есть по умолчанию приложение в этом образе крутится от рута, если в нем найдут RCE и атакующий вылезет наружу через docker-уязвимость или примонтированный volume - получит root-доступ к хосту

Скрин с тремя whoami лежит рядом: images/part1_whoami.png

Часть II

Написал свой Dockerfile. Файл лежит рядом (./Dockerfile)

Сборка и проверка
```
docker build -t my-secure-rest .
docker run -it --entrypoint sh my-secure-rest
whoami
```
Внутри теперь возвращается appuser. Скрин - images/part2_whoami_appuser.png

В Dockerfile в комментариях напишу какие еще настройки безопасности нужны

Часть III

Прогнал образ через trivy:
```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
    aquasec/trivy:latest image --severity HIGH,CRITICAL --no-progress ket9/otus-devsecops-owasp-rest:latest
```
Скрин с выводом - images/part3_trivy.png

Что увидел в выводе:
- образ собран на старой Alpine-базе 3.14.2, которую дистрибутив уже не поддерживает
- по итогу trivy насчитал 70 HIGH/CRITICAL CVE: 56 по Alpine-пакетам и 14 по Python-пакетам
- Python-зависимости приложения тоже подсвечены, старые версии с известными CVE
- секретов trivy в этом образе не нашел

Выводы про безопасность образа:
- образ устаревший и уязвимый, для прод-использования не годится без пересборки
- основная проблема - старая ОС-база и старые Python-зависимости; лечится переходом на актуальный slim/alpine-образ и обновлением requirements.txt
- образ запускает приложение от root - это то что я уже починил в Части II
- нужнно встроить trivy в CI и блокировать пуш образа в registry если есть HIGH/CRITICAL без явного исключения через .trivyignore