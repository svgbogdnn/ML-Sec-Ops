FROM ket9/otus-devsecops-xss:latest

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

EXPOSE 80

USER appuser

# Дополнительные меры безопасности:
# 1. Запускать контейнер с read-only filesystem:
#    docker run --read-only ...
# 2. Ограничить capabilities:
#    docker run --cap-drop=ALL ...
# 3. Запретить повышение привилегий:
#    docker run --security-opt no-new-privileges ...
# 4. Ограничить ресурсы:
#    docker run --memory=256m --cpus=0.5 ...
# 5. Пробрасывать только необходимые порты:
#    docker run -p 8080:80 ...
