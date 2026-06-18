# Лабораторная работа №5

## Часть 1

Образ ket9/otus-devsecops-owasp-rest:latest.
На хосте:  
whoami  
Ответ: laptop-72p9afnc\artmk.  
sh внутри контейнера:  
docker run -it --rm --entrypoint /bin/sh ket9/otus-devsecops-owasp-rest:latest  
Внутри:  
whoami  
Ответ: root.  
По умолчанию процессы в контейнере работают от root.  

## Часть 2

Создал Dockerfile в C:\Users\artmk\Desktop\Lab5:
```
FROM ket9/otus-devsecops-owasp-rest:latest
EXPOSE 8080
RUN adduser -D -H -s /bin/sh appuser
USER appuser
# Безопасный запуск:
# --read-only
# --cap-drop=ALL
# --security-opt=no-new-privileges
# --memory=256m --cpus=1
# не использовать --privileged
```
Сборка: docker build -t mysecureapp .  
Запуск с проверкой:  
docker run -it --rm --entrypoint /bin/sh mysecureapp  
whoami  
Ответ: appuser. Пользователь не root.  

## Часть 3

Сканирование Trivy:  
docker run --rm -v //var/run/docker.sock:/var/run/docker.sock -v $HOME/.cache/trivy:/root/.cache/ aquasec/trivy:latest image mysecureapp  

Найдено 72 уязвимости в системных пакетах (alpine 3.14.2) и 41 в Python-библиотеках. Из них 8 критических (zlib, expat), 48 высоких, 16 средних. Критические позволяют выполнить произвольный код или вызвать переполнение буфера.  
Вывод: базовый образ содержит известные уязвимости. Нужно обновлять образы, использовать не-root пользователя и флаги безопасности, регулярно сканировать образы.  