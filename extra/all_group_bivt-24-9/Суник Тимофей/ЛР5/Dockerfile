FROM ket9/otus-devsecops-owasp-rest:latest
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
EXPOSE 8080

# - не использовать latest (фиксированный тег)
# - запуск от non-root пользователя
# - минимизация слоев и пакетов
# - использовать --read-only при запуске
# - ограничивать ресурсы (--memory, --cpus)
# - отключать лишние capability (--cap-drop=ALL)
# - использовать seccomp/apparmor профили