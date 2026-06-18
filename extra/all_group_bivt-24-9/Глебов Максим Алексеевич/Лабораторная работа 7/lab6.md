# Лабораторная работа 6

**Тема:** поиск секретов в репозитории git с помощью truffleHog.

---

# Цель

Проверить репозиторий на наличие секретов и прислать аналитику по найденным секретам.

Репозиторий:

```text
https://github.com/OtusTeam/DevSecOps_secret-finding.git
```

---

# Как сканировал

Запускал truffleHog в одноразовом Docker-контейнере с Python, как в методичке:

```bash
docker run -it -p 8080:8080 python:buster /bin/bash
pip install truffleHog
trufflehog https://github.com/OtusTeam/DevSecOps_secret-finding.git | tee -a log.txt
```

truffleHog проходит по всей истории коммитов всех веток и ищет секреты двумя способами: по регулярным выражениям и по энтропии Шеннона (длинные строки base64/hex с высокой энтропией).

В ветке `main` лежит только `README.md`. Настоящие секреты спрятаны в истории (около 2600 коммитов) и в ветке `develop`, поэтому нужен именно анализ истории git, а не текущих файлов.

Часть срабатываний оказалась ложной (например, checksum'ы зависимостей). Ниже 6 настоящих секретов разных типов.

---

# Часть 1. Найденные секреты

## Секрет 1. AWS Access Key

**Расположение:**  
`.travis.yml`, строки 24-25.

```yaml
deploy:
  provider: s3
  access_key_id: AKIAJQLKPGHXRH2AH5QA
  secret_access_key:
    secure: 45+SwWlPFujD9FOOFLA9Lz0CaePVrn/SEsAhAn0Ve9sYpI0Vsij...
  bucket: webgoat-war
```

**Почему это секрет:**  
`AKIAJQLKPGHXRH2AH5QA` это AWS Access Key ID (виден по префиксу `AKIA` и длине 20 символов). Вместе с `secret_access_key` он даёт доступ к аккаунту AWS и бакету `webgoat-war`.

**Как правильно хранить:**  
Не держать ключи в коде. Использовать секреты CI или временные роли (IAM Roles, OIDC). Утёкший ключ сразу отозвать.

---

## Секрет 2. Приватный ключ

**Расположение:**  
`webgoat-server/privatekey.key`, строка 1.

```text
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCpuOH6AnMc/xdJ
...
-----END PRIVATE KEY-----
```

**Почему это секрет:**  
Это закрытый ключ, видно по заголовку `-----BEGIN PRIVATE KEY-----`. С ним можно расшифровывать трафик и выдавать себя за сервер.

**Как правильно хранить:**  
Хранить в хранилище секретов (Vault, AWS KMS) или в keystore. В git добавить `*.key`, `*.pem` в `.gitignore`. Утёкший ключ перевыпустить.

---

## Секрет 3. Пароль БД в строке подключения

**Расположение:**  
`docker-compose-postgres.yml`, строки 10 и 19.

```yaml
SPRING_DATASOURCE_URL: jdbc:postgresql://webgoat_db:5432/webgoat?user=webgoat&password=webgoat
...
postgresql://webgoat:webgoat@webgoat_db:5432
```

**Почему это секрет:**  
В строке подключения в открытом виде записаны логин и пароль к базе `webgoat:webgoat`. С ними можно подключиться к БД.

**Как правильно хранить:**  
Логин и пароль выносить в переменные окружения или секреты (Docker secrets, Kubernetes Secrets). Пароль по умолчанию сменить.

---

## Секрет 4. Учётные данные в URL

**Расположение:**  
`webgoat.sh`, строка 49.

```bash
curl http://guest:guest@127.0.0.1:8080/WebGoat/attack
```

**Почему это секрет:**  
Логин и пароль `guest:guest` записаны прямо в URL. Такие данные попадают в логи и историю команд.

**Как правильно хранить:**  
Не передавать пароль в URL. Использовать заголовок `Authorization`, файл `~/.netrc` или переменные окружения.

---

## Секрет 5. JWT-токен

**Расположение:**  
`webgoat-lessons/jwt/.../lessonPlans/en/JWT_signing_solution.adoc`, строка 35.

```text
access_token=eyJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE2MDgxMjg1NjYsImFkbWluIjoiZmFsc2UiLCJ1c2VyIjoiVG9tIn0.rTSX6PSXqUoGUvQQDBiqX0re2BSt7s2-X6FPf34Qly9SMpqIUSP8jykedJbjOBNlM3_CTjgk1SvUv48Pz8zIzA
```

**Почему это секрет:**  
Это подписанный JWT (алгоритм `HS512`). В payload лежит `{"iat":1608128566,"admin":"false","user":"Tom"}`. По токену можно авторизоваться от имени пользователя.

**Как правильно хранить:**  
JWT нельзя класть в репозиторий, документацию и логи. Токены делать короткоживущими, ключ подписи хранить в секретах.

---

## Секрет 6. Пароли пользователей в открытом виде

**Расположение:**  
`webgoat-lessons/sql-injection/.../db/migration/V2019_09_26_6__user_system_data.sql`, строки 8-10.

```sql
INSERT INTO user_system_data VALUES (101,'jsnow','passwd1', '');
INSERT INTO user_system_data VALUES (102,'jdoe','passwd2', '');
INSERT INTO user_system_data VALUES (103,'jplane','passwd3', '');
```

**Почему это секрет:**  
Пароли пользователей записаны в открытом виде (`passwd1`, `passwd2`, `passwd3`). Пароли в БД нельзя хранить в plaintext.

**Как правильно хранить:**  
Хранить только хеш с солью (`bcrypt`, `argon2`, `scrypt`, `PBKDF2`). В тестовых данных не использовать настоящие пароли.

---

# Часть 2. Дополнительно

## BFG Repo-Cleaner

BFG Repo-Cleaner это инструмент для быстрой очистки истории git от больших файлов и секретов. Он проще и быстрее, чем `git filter-branch`.

Зачем нужен: если секрет попал в git, удалить его из последнего коммита мало, он остаётся в истории. BFG переписывает всю историю и заменяет секреты:

```bash
bfg --replace-text passwords.txt my-repo.git
bfg --delete-files privatekey.key my-repo.git
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

После этого меняются хеши коммитов, нужен `force push`, а всем участникам переклонировать репозиторий. Утёкший секрет всё равно нужно сменить.

## Можно ли автоматизировать поиск секретов

Да, поиск секретов автоматизируют на нескольких этапах:

- pre-commit hook (`gitleaks`, `truffleHog`, `detect-secrets`), секрет не попадёт в репозиторий;
- шаг сканирования в CI/CD, при находке сборка падает;
- GitHub Secret Scanning и push protection на стороне платформы.

Это правильный подход, но с оговорками:

- лучше ловить секрет до попадания в историю, после утечки чистить дорого;
- сканеры дают ложные срабатывания, нужен allow-list и ручной разбор;
- главное не хранить секреты в коде вообще, а сканирование это страховка.

---

# Итог

С помощью truffleHog по истории всех веток найдено 6 секретов разных типов: AWS Access Key, приватный ключ, пароль БД в строке подключения, учётные данные в URL, JWT-токен и пароли пользователей в plaintext. Часть срабатываний оказалась ложной.

Главный вывод: секреты нельзя хранить в коде и истории git. Их место в хранилищах секретов и переменных окружения. Утёкшие секреты нужно менять, историю чистить (BFG Repo-Cleaner), а поиск секретов встраивать в pre-commit и CI/CD.
