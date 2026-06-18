
FROM ket9/otus-devsecops:latest

EXPOSE 8080

USER root

RUN groupadd -r appgroup && useradd -r -g appgroup appuser


WORKDIR /app
RUN chown -R appuser:appgroup /app

USER appuser

CMD ["tail", "-f", "/dev/null"]



# КОММЕНТАРИИ ПО БЕЗОПАСНОСТИ (для Части II):
#
# Настройки в Dockerfile:
# 1. HEALTHCHECK — проверка здоровья приложения, чтобы вовремя убить зависший контейнер.
# 2. Удаление ненужных утилит (curl, wget, git) после сборки, чтобы у хакера было меньше инструментов.
#
# Флаги запуска (указываются в docker run или docker-compose):
# - read_only: true — файловая система только для чтения.
# - security_opt: ["no-new-privileges:true"] — запрет на повышение прав[cite: 1].
# - cap_drop: ["ALL"] — сброс всех потенциально опасных привилегий ядра.