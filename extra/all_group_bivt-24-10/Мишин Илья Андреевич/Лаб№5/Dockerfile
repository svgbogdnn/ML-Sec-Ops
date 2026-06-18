FROM python:3.11-slim

RUN groupadd -r testgr && \
    useradd -r -g testgr test

USER test

CMD ["python3", "--version"]

# Дополнительные настройки безопасности:
# 1. Запускать с read-only файловой системой:
#    docker run --read-only
# 2. Ограничить cpu и memory:
#    docker run --memory=512m --cpus=1
# 3. Запретить повышение привилегий:
#    docker run --security-opt=no-new-privileges:true
# 4. Ограничить capabilities:
#    docker run --cap-drop=ALL
