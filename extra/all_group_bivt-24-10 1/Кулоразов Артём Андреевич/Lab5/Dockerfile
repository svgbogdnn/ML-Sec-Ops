FROM ket9/otus-devsecops-owasp-rest:latest
EXPOSE 8080
RUN adduser -D -H -s /bin/sh appuser
USER appuser
# Безопасный запуск:
# --read-only
# --cap-drop=ALL
# --security-opt=no-new-privileges
# --memory=256m --cpus=1
# не использовать --privileged