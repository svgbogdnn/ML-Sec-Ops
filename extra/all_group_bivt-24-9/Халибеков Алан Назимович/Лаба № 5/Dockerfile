FROM postgres:16-alpine

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN mkdir -p /home/appuser /tmp/app /var/lib/postgresql/data /var/run/postgresql \
    && chown -R appuser:appgroup /home/appuser /tmp/app /var/lib/postgresql /var/run/postgresql

ENV PGDATA=/var/lib/postgresql/data

USER appuser

# дополнительные настройки безопасности при запуске контейнера:
# docker run --read-only --cap-drop ALL --security-opt no-new-privileges
# docker run --memory 256m --cpus 1
# docker run -p 127.0.0.1:5432:5432
# также лучше не передавать секреты через аргументы командной строки
