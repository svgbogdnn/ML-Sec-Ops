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