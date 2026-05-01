FROM productivity-app-mas-taska-server:latest

RUN addgroup --system testgr && \
    adduser --system --ingroup testgr test

USER test

CMD ["python3", "app.py"]

# Дополнительные настройки:
# 1. Запускать с read-only файловой системой:
#    docker run --read-only ...
# 2. Ограничить cpu и memrory:
#    docker run --memory=512m --cpus=1 ...
# 3. Ограничить capabilities:
#    docker run --cap-drop=ALL ...