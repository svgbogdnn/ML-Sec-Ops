FROM ket9/otus-devsecops-owasp-rest:latest
RUN addgroup -S appgroup 2>/dev/null || groupadd -r appgroup; \
    adduser -S -G appgroup -h /home/appuser appuser 2>/dev/null || useradd -r -g appgroup -m -d /home/appuser appuser

USER appuser

EXPOSE 8080


# В самом Dockerfile:
#   - взять образ полегче (alpine / distroless / scratch)
#   - вместо latest написать точную версию или хеш
#   - multi-stage: сначала собрать, потом взять только готовый бинарь
#   - копировать файлы с указанием владельца: COPY --chown=appuser:appgroup;
#   - чистить за собой кеш пакетов в той же команде:
#       apt-get update && apt-get install -y --no-install-recommends X \
#         && rm -rf /var/lib/apt/lists/*;
#   - добавить HEALTHCHECK, чтобы проверять, жив ли контейнер;
#   - не хранить секреты в ENV/ARG, потому что они останутся в истории слоёв —
#     лучше через docker secrets или BuildKit --mount=type=secret;
#   - сделать .dockerignore, чтобы случайно не утащить .git, .env, ключи и тесты.
#
# При запуске контейнера:
#   docker run \
#     --read-only \                       # файловую систему на чтение
#     --tmpfs /tmp:rw,size=64m,mode=1777 \ # если приложению нужна временная папка
#     --cap-drop=ALL \                    # отключаем всё лишнее
#     --security-opt=no-new-privileges \  # запрещаем повышать права
#     --pids-limit=200 \                  # ограничиваем количество процессов
#     --memory=512m --cpus=0.5 \          # чтобы не было утечки за лимиты
#     --network=app-net \                 # своя сеть, а не общая
#     -p 127.0.0.1:8080:8080 \            # localhost
#     --user 1000:1000 \                  # запускаем от обычного пользователя
#     -d image
#   - подписать образ (cosign / Docker Content Trust);
#   - проверить образ анализатором вроде trivy/grype
#   - не подключать /var/run/docker.sock внутрь контейнера;
#   - использовать seccomp/AppArmor профили (--security-opt seccomp=...);
#   - пересобирать образ время от времени, чтобы подтянуть свежие обновления.
