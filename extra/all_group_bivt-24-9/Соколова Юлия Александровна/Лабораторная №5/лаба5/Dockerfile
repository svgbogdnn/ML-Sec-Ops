# Берём базовый образ из Docker Hub 
FROM bash:4.3.48-alpine3.22

# Создаём нового пользователя appuser (не root)
RUN adduser -D appuser

# НАСТРОЙКИ БЕЗОПАСНОСТИ В DOCKERFILE

# 1. USER — запускаем прилsожение не от root.
#    Если злоумышленник вырвется из приложения, он получит минимальные права.
USER appuser

# 2. HEALTHCHECK — Docker будет проверять, живо ли приложение.
#    Если оно зависнет, контейнер получит статус unhealthy.
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:8080/ || exit 1

# 3. Не хранить секреты (пароли, токены) в ENV или ARG —
#    они видны через docker inspect и в истории образа.
#    Использовать docker secrets или внешние vault-решения.

EXPOSE 8080