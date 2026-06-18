FROM ket9/otus-devsecops-owasp-rest:latest

RUN adduser -D -H -s /sbin/nologin appuser

USER appuser

EXPOSE 8080

# Дополнительные настройки безопасности при запуске контейнера:
# docker run --rm --read-only --security-opt no-new-privileges \
#   --cap-drop ALL --pids-limit 100 --memory 256m \
#   --cpus 0.5 --tmpfs /tmp:rw,noexec,nosuid,size=64m \
#   lab6-secure
#
# Что это дает:
# --read-only запрещает запись в файловую систему контейнера;
# --security-opt no-new-privileges запрещает повышение привилегий;
# --cap-drop ALL убирает Linux capabilities, которые не нужны приложению;
# --pids-limit, --memory и --cpus ограничивают ресурсы;
# --tmpfs /tmp дает временную директорию без исполнения файлов.
