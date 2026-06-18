
FROM alpine:latest
LABEL maintainer="student"
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -s /bin/sh -D appuser
USER appuser
WORKDIR /home/appuser

CMD ["echo", "Контейнер запущен"]

# Дополнительные настройки
# 1. --read-only
#    делает файловую систему только для чтения
#    пример: docker run --read-only my_secure_app
#
# 2. --cap-drop=ALL
#    убирает все лишние привилегии ядра
#    пример: docker run --cap-drop=ALL my_secure_app
#
# 3. --security-opt no-new-privileges:true
#    запрещает процессам повышать свои права внутри контейнера
#
# 4. ограничение ресурсов (--memory, --cpus)
#    защищает сервер от зависания
