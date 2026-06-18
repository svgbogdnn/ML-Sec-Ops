# Лабораторная работа: Базовая настройка безопасности Docker-контейнера

## Часть I: Проверка пользователя по умолчанию

### Результат

**На хосте:**
```bash
$ whoami
user
```

**Внутри контейнера:**
```bash
$ docker run -it alpine:3.19 sh
/ # whoami
root
```
**Вывод:** Контейнеры по умолчанию запускаются от `root` — это риск безопасности.

---

## Часть II: Создание Dockerfile с non-root пользователем

### Dockerfile

```dockerfile
FROM alpine:3.19

# Создаём non-root пользователя
RUN addgroup -g 1000 appgroup && \
    adduser -D -u 1000 -G appgroup appuser

WORKDIR /app
RUN chown -R appuser:appgroup /app

# Переключаемся на non-root пользователя
USER appuser

EXPOSE 8080
CMD ["sh", "-c", "whoami && id && echo 'SUCCESS: Running as non-root user!'"]

# ДОП. МЕРЫ БЕЗОПАСНОСТИ (комментарии):
# --read-only --no-new-privileges --cap-drop=ALL
# --memory 512m --cpus 1.0
# -p 127.0.0.1:8080:8080
```

### Результат запуска

```bash
$ docker build -t my-secure-app:latest .
$ docker run -it my-secure-app:latest
appuser
uid=1000(appuser) gid=1000(appgroup)
SUCCESS: Running as non-root user!
```

**Вывод:** Контейнер успешно запускается от `appuser`, а не от `root`.

---

## Часть III: Сканирование образа через Trivy

### Результат сканирования

![Сканирование Trivy](image-1.png)

![Детали уязвимостей](image-2.png)

### Выводы

- Нет критических уязвимостей
-  Обнаружены уязвимости высокой и средней степени
- Образ использует minimal Alpine + non-root пользователь

```