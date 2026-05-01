FROM ket9/otus-devsecops-owasp-rest:latest

# Создаём отдельного пользователя без root-прав.
# UID/GID 10001 выбран явно, чтобы процесс в контейнере не работал от root.
RUN echo "appuser:x:10001:10001:appuser:/home/appuser:/bin/sh" >> /etc/passwd && \
    echo "appgroup:x:10001:" >> /etc/group && \
    mkdir -p /home/appuser && \
    chown -R 10001:10001 /home/appuser

# Открываем порт приложения
EXPOSE 8080

# Запускаем контейнер не от root
USER 10001:10001

# Дополнительные настройки безопасности, которые можно использовать при запуске:
#
# 1. Не использовать --privileged, потому что он дает контейнеру слишком много прав.
#
# 2. Запрещать повышение привилегий:
#    --security-opt=no-new-privileges
#
# 3. Убирать лишние capabilities:
#    --cap-drop=ALL
#
# 4. Делать файловую систему контейнера read-only:
#    --read-only
#
# 5. Ограничивать количество процессов:
#    --pids-limit=100
#
# 6. Ограничивать память:
#    --memory=512m
#
# 7. Не монтировать /var/run/docker.sock внутрь обычного приложения.
#
# Пример запуска:
# docker run --rm -p 8080:8080 \
#   --read-only \
#   --cap-drop=ALL \
#   --security-opt=no-new-privileges \
#   --pids-limit=100 \
#   --memory=512m \
#   secure-owasp-rest:1.0
