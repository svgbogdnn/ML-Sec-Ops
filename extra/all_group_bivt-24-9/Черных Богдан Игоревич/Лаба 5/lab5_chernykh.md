# Лабораторная Работа №5

*Черных Богдан Игоревич ' БИВТ-24-9*
ㅤ
ㅤ

# 🌐 Часть 1

#### Условие -

*"скачать любой docker-образ из репозитория docker.io (можно использовать любой из предыдущих домашних заданий)
с помощью команды docker image ls посмотреть, какие образы есть на вашей машине
до запуска контейнера выполнить команду whoami
запустить командную оболочку sh внутри выбранного контейнера с помощью команды docker run -it <имя_образа_из_пункта_2> sh
внутри контейнера выполнить команду whoami и посмотреть пользователя. Сделать скриншот с пунктами 3-5, он и будет ответом на первую часть домашнего задания."*
ㅤ
ㅤ
Скачал docker образ и посмотрел список образов через `docker image ls`.

До запуска контейнера выполнил `whoami`. Ответ - `svg\lenovo`.

Запустил контейнер с явным указанием начальной точки:

```sh
docker run -it --entrypoint sh ket9/otus-devsecops-owasp-rest:latest
```

Внутри контейнера выполнил `whoami`. Ответ - `root`. Значит исходный контейнер запускается от пользователя `root`, что плохо с точки зрения безопасности.

### ✔️ Скриншот с пунктами 3-5 в папке с названием `**part'1.png**`

ㅤ

# 🌐 Часть 2

#### Условие -

*"Напишите простейший dockerfile, который выполнит build контейнера на основе базового контейнера из части 1 (например,
FROM ket9/otus-devsecops-owasp-rest:latest
EXPOSE 8080)
дополните dockerfile необходимыми инструкциями, чтобы внутри контейнера приложение запускалось не от пользователя root
зайдите внутрь контейнера и выполните whoami.
Подумайте, какие еще настройки безопасности можно указать в самом dockerfile и с какими флагами запускать контейнер.
Укажите это в комментарии внутри Dockerfile.
Ответом ко второй части задания будет написанный Dockerfile и скриншот из пункта 2."*
ㅤ
ㅤ
Создал `Dockerfile` на основе исходного образа (`FROM ket9/otus-devsecops-owasp-rest:latest`). Сам файл как есть в папке, так и ниже.

```dockerfile
#Dockerfile
FROM ket9/otus-devsecops-owasp-rest:latest

LABEL lab="basic-docker-container-security"
LABEL description="Secure variant of OWASP REST container with non-root user"

ARG APP_USER=appuser
ARG APP_GROUP=appgroup
ARG APP_UID=10001
ARG APP_GID=10001

USER root

ENV HOME=/home/appuser
ENV TMPDIR=/tmp/app
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

SHELL ["/bin/sh", "-c"]

RUN set -eux; \
    if command -v groupadd >/dev/null 2>&1; then \
        groupadd --gid "${APP_GID}" "${APP_GROUP}"; \
        useradd \
            --uid "${APP_UID}" \
            --gid "${APP_GID}" \
            --create-home \
            --home-dir "/home/${APP_USER}" \
            --shell /usr/sbin/nologin \
            "${APP_USER}"; \
    else \
        addgroup -g "${APP_GID}" "${APP_GROUP}"; \
        adduser \
            -D \
            -u "${APP_UID}" \
            -G "${APP_GROUP}" \
            -h "/home/${APP_USER}" \
            -s /sbin/nologin \
            "${APP_USER}"; \
    fi; \
    mkdir -p /tmp/app /var/tmp/app; \
    chown -R "${APP_UID}:${APP_GID}" "/home/${APP_USER}" /tmp/app /var/tmp/app; \
    chmod 700 "/home/${APP_USER}"; \
    chmod 1777 /tmp/app /var/tmp/app

WORKDIR /home/appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD python -c "import socket; s=socket.create_connection(('127.0.0.1', 8080), 2); s.close()" || exit 1

USER 10001:10001

# Дополнительные настройки безопасности при запуске контейнера:
# "Подумайте, какие еще настройки безопасности можно указать в самом dockerfile и с какими флагами запускать контейнер. Укажите это в комментарии внутри Dockerfile."

# docker run --rm -p 8080:8080 --read-only --tmpfs /tmp:rw,noexec,nosuid,size=64m --cap-drop=ALL --security-opt=no-new-privileges --pids-limit=100 --memory=256m --cpus=0.5 secure-owasp-rest
#
# --read-only                       делает файловую систему контейнера доступной только для чтения
# --tmpfs /tmp                      дает контейнеру отдельную временную директорию для записи
# --cap-drop=ALL                    убирает лишние Linux capabilities
# --security-opt=no-new-privileges  запрещает повышение привилегий внутри контейнера
# --pids-limit=100                  ограничивает количество процессов
# --memory=256m                     ограничивает потребление памяти
# --cpus=0.5                        ограничивает использование CPU
# --privileged использовать нельзя т.к. он дает контейнеру слишком широкие права
```

ㅤ
В `Dockerfile` был добавлен отдельный пользователь `appuser`, после чего контейнер был переключен на запуск от непривилегированного пользователя `USER 10001:10001`. Смысл - контейнер больше не должен запускаться от лица `root`.

После докерфайла собрал новый образ -

```sh
docker build -t secure-owasp-rest
```

Запустил контейнер -

```sh
docker run --rm -it --entrypoint sh secure-owasp-rest
```

Внутри контейнера `whoami`. Ответ - `appuser`. Это подтверждает что новый контейнер запускается не от `root` (**что и требовалось сделать**), а от отдельного непривилегированного пользователя.

### ✔️ Скриншот из пункта 2 в папке с названием `**part'2.png`**

### ✔️ Dockerfile в папке с названием  `**Dockerfile`**

ㅤ
ㅤ

# 🌐 Часть 3

#### Условие -

*"Прогнать образ через один из сканеров безопасности:
[https://github.com/quay/clair](https://github.com/quay/clair)
[https://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)
Проанализировать его вывод. Ответ - ваши выводы про безопасность выбранного образа, а также скрин с выводом сканера."*
ㅤ
ㅤ
Просканировал собранный образ `secure-owasp-rest` с помощью сканера безопасности `Trivy`.

Для проверки использовалась команда -

```sh
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --scanners vuln --severity HIGH,CRITICAL --ignore-unfixed --format table secure-owasp-rest
```

Здесь оставил только уязвимости уровня `HIGH` и `CRITICAL` чтобы отчет был хотя бы немного читаемым.

Trivy определил что образ основан на `Alpine 3.14.2`.

Также было предупреждение что версия `Alpine 3.14.2` больше не поддерживается дистрибутивом, поэтому обнаружение уязвимостей может быть неполным т.к. security updates уже не предоставляются.
ㅤ
ㅤ
В общем отчете Trivy показал, что для системной части образа найдено -

```sh
secure-owasp-rest (alpine 3.14.2)
Total: 56
HIGH: 48
CRITICAL: 8
```

То есть в системных пакетах базового образа найдено **56 уязвимостей высокого и критического уровня**. Среди уязвимых компонентов например `busybox`, `expat`, `zlib`, `libcrypto`, `libssl`.
ㅤ
ㅤ
Также Trivy отдельно проверил Python-зависимости приложения. В блоке `Python (python-pkg)` было найдено:

```sh
Total: 14
HIGH: 14
CRITICAL: 0
```

Среди уязвимых питон пакетов были `Flask`, `PyJWT`, `Werkzeug`, `certifi`, `setuptools`, `urllib3`, `wheel`.  
ㅤ  
ㅤ  
### 🔹 **Вывод**

По результатам сканирования видно что образ `secure-owasp-rest` **нельзя считать безопасным**. Несмотря на то что во второй части контейнер был переведен на запуск от пользователя `appuser`, внутри образа остались уязвимые системные пакеты и питон зависимости.

**Главная проблема** связана с устаревшей базовой системой `Alpine 3.14.2` которая *больше не поддерживается*. Из-за этого образ содержит большое количество известных CVE, включая уязвимости уровня **HIGH** и **CRITICAL**. Кроме того, приложение использует старые версии питон библиотек, например `Flask`, `PyJWT`, `Werkzeug`, `setuptools` и `urllib3`, для которых `Trivy` также нашел уязвимости.

Итоговый вывод: запуск контейнера не от `root` улучшает безопасность - но этого, как видно, **недостаточно**. Безопасность Docker образа зависит не только от пользователя внутри контейнера - но и от актуальности базовой ОС, системных пакетов и зависимостей приложения. Для улучшения безопасности нужно обновить базовый образ, использовать поддерживаемую версию `Alpine` или другой актуальный образ, обновить питон зависимости до исправленных версий, пересобрать контейнер и **регулярно** повторять сканирование через `Trivy`.

### ✔️ Скриншоты с выводом сканера в папке с названиями **`part'3_1.png`**, **`part'3_2.png`**, **`part'3_3.png`**.