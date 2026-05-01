FROM ket9/otus-devsecops-owasp-rest:latest

RUN adduser -D -h /home/appuser -s /bin/sh appuser
RUN chown -R appuser:appuser /vampi/database /tmp

USER appuser

# допом можно добавить ограничения по ресурсам
# --memory 256m 
# --cpus 1
# или только чтение
# --read-only
# или убрать вообще все привилегии
# --cap-drop ALL
# открыть один порт
# -p 127.0.0.1:8080:8080