# ЛР 6. Поиск секретов в репозитории

Усенко Тимофей БИВТ-24-10

## Что делал

По методичке запустил `trufflehog` в Docker-контейнере на репозитории `https://github.com/OtusTeam/DevSecOps_secret-finding` (ветка `develop`).

Команды:

```bash
git clone --depth=50 -b develop https://github.com/OtusTeam/DevSecOps_secret-finding.git repo

docker run --rm -v ~/Desktop/hw_devsecops_lr6:/work -w /work python:buster \
    /bin/bash -c "pip install truffleHog && trufflehog --regex --entropy=True file:///work/repo > log.txt 2>&1"
```

Вывод сохранил в `log.txt`. Trufflehog подсветил много мест, разобрал 5 разных по типу секретов.

## Найденные секреты

### 1. GitHub токен в CI

Файл: `.github/workflows/release.yml`, строка 75

```
GITHUB_TOKEN: cee3257f0e59d1d4975a35429cffc3d8b5fabe15ca2e9d47243cbc446d4894a5
```

Почему секрет: токен GitHub лежит прямо в коде. С ним можно делать что угодно от имени проекта — пушить, создавать релизы.

Как надо: положить в GitHub Secrets и использовать как `${{ secrets.GITHUB_TOKEN }}`.

### 2. Приватный RSA-ключ

Файл: `webgoat-server/privatekey.key`

```
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCpuOH6AnMc/xdJ
...
-----END PRIVATE KEY-----
```

Почему секрет: это приватный ключ. С ним можно расшифровывать трафик и подписывать что угодно от имени сервера. В репозитории ему не место.

Как надо: хранить в секрет-менеджере (Vault, AWS Secrets Manager) или GitHub Secrets. На сервере — файл с правами 600. В репо можно класть только публичный ключ.

### 3. Пароль в коде

Файл: `webgoat-lessons/challenge/src/main/java/org/owasp/webgoat/challenges/SolutionConstants.java`, строка 35

```java
String PASSWORD_TOM = "thisisasecretfortomonly";
```

Почему секрет: пароль пользователя зашит в исходник. Любой, у кого есть доступ к репо, его знает.

Как надо: читать пароли из переменных окружения или конфига. В базе хранить только хеш (bcrypt/argon2).

### 4. Admin-токен в коде (похож на MD5)

Файл: тот же `SolutionConstants.java`, строка 36

```java
String ADMIN_PASSWORD_LINK = "375afe1104f4a487a73823c50a9292a2";
```

Почему секрет: 32-символьный hex, по имени переменной — какая-то админская секретная ссылка. Зашита в коде, поэтому больше не секретная.

Как надо: генерировать на лету через `SecureRandom`/UUID. Если нужно хранить — то в секрет-менеджере, и ставить срок жизни.

### 5. CSR (запрос на сертификат)

Файл: `webgoat-server/certificatesigningrequest.csr`

```
-----BEGIN CERTIFICATE REQUEST-----
MIIC4zCCAcsCAQAwdDELMAkGA1UEBhMCNDQxCzAJBgNVBAgMAmxsMQ4wDAYDVQQH
...
-----END CERTIFICATE REQUEST-----
```

Почему секрет: внутри есть данные о владельце (email `test@test.ru`, организация) и публичный ключ. Сам по себе не катастрофа, но лежит рядом с приватным ключом — значит к секретам отношение плохое.

Как надо: после получения сертификата CSR не нужен, его можно удалить. В репо не коммитить.

## Дополнительно

**BFG Repo-Cleaner** — утилита для чистки истории git. Если секрет уже закоммитили, простое удаление не помогает — он остаётся в истории. BFG проходит по всем коммитам и удаляет/заменяет нужные строки или файлы. После чистки нужен force push и обязательная замена всех скомпрометированных секретов.

**Автоматизация поиска** — нужная штука. Можно ставить `trufflehog`/`gitleaks` в pre-commit хук и в CI, чтобы блокировать пуш если в коде появился секрет. Но есть минусы: много ложных срабатываний (видно по нашему логу — куча шума на checksum-ах и сжатом JS), поэтому нужно настраивать allowlist. Сканер сам по себе не решает проблему — параллельно должен быть нормальный секрет-менеджер и культура работы с секретами на код-ревью.
