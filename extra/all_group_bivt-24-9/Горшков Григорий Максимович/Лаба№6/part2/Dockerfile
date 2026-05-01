FROM ket9/otus-devsecops-owasp-rest:latest

RUN adduser -D -h /home/appuser -s /bin/sh appuser
RUN chown -R appuser:appuser /vampi/database /tmp

USER appuser

# Дополнительные настройки безопасности:
# - запускать контейнер с ограничением ресурсов: --memory 256m --cpus 1
# - использовать файловую систему только для чтения: --read-only
# - добавить tmpfs для временных файлов: --tmpfs /tmp:rw,noexec,nosuid,nodev
# - убрать лишние Linux capabilities: --cap-drop=ALL
# - запретить повышение привилегий: --security-opt=no-new-privileges
# - не запускать контейнер с флагом --privileged
# - публиковать порт только на localhost: -p 127.0.0.1:8080:8080
