FROM alpine:latest

# Создаем отдельную группу и пользователя без root-прав
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Задаем рабочую директорию внутри контейнера
WORKDIR /app

# Даем созданному пользователю права на рабочую директорию
RUN chown -R appuser:appgroup /app

# Указываем, что приложение может использовать порт 8080
EXPOSE 8080

# Запускаем контейнер не от root, а от обычного пользователя
USER appuser

# Дополнительные флаги безопасности при запуске контейнера:
# --cap-drop=ALL - убрать все лишние capabilities
# --network=none - отключить сеть, если контейнеру она не нужна
# --security-opt=no-new-privileges:true - запретить повышение привилегий