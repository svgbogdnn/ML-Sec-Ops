FROM ket9/otus-devsecops-xss:latest
EXPOSE 8080

RUN groupadd -g 1001 appgroup && useradd -u 1001 -g appgroup -m appuser
USER appuser

# Дополнительные меры безопасности:
# 1. Флаги для docker run:
#    --read-only                    # только для чтения
#    --tmpfs /tmp --tmpfs /var/run  # временные директории в памяти
#    --no-new-privileges:true       # запретить повышение привилегий
#
# 2. Ограничение ресурсов:
#    --memory="512m" --cpus="1.0" --pids-limit=100
#
# 3. Сеть:
#    --network=bridge (не host), или создать кастомную сеть
#
# 4. Никогда не монтировать:
#    -v /var/run/docker.sock  # доступ к демону Docker = root на хосте
#    -v /:/host               # доступ ко всей ФС хоста
#
# 5. Регулярно обновлять базовый образ и сканировать на уязвимости