# ЛР 6

Я прописала следующие команды, чтобы запустить сканирование репозитория
```bash
docker run -it -p 8080:8080 python:buster /bin/bash

pip install truffleHog

trufflehog https://github.com/OtusTeam/DevSecOps_secret-finding.git | tee -a log.txt
```

## Найденные секреты

### 1 GitHub Personal Access Token

[пример](./image_1.png)

| Параметр | Значение |
|----------|----------|
| **Расположение** | `.github/workflows/release.yml`, строка с `GITHUB_TOKEN:` |
| **Фрагмент** | `cee3257f0e59d1d4975a35429cffc3d8b5fabe15ca2e9d47243cbc446d4894a5` |
| **Почему секрет** | 64 hex-символа, высокая энтропия, формат GitHub PAT. Даёт доступ к репозиториям и CI/CD. |
| **Как хранить** | GitHub Secrets `${{ secrets.GITHUB_TOKEN }}`. Не коммитить в код. |

### 2 Private Key (SSL/TLS)
[пример](./image2.png)
| Параметр | Значение |
|----------|----------|
| **Расположение** | `webgoat-server/privatekey.key` |
| **Фрагмент** | -----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQC7... |
| **Почему секрет** | Приватный ключ позволяет расшифровывать трафик, подписывать код, аутентифицироваться. |
| **Как хранить** | HashiCorp Vault или AWS Secrets Manager. В коде - только ссылки. Права `600`. |

### 3 Serialized Java Token
[пример](./image3.png)
| Параметр | Значение |
|----------|----------|
| **Расположение** | `src/test/java/.../InsecureDeserializationTest.java` |
| **Фрагмент** | `rO0ABXNyADFvcmcuZHVja3R5cGVzZXJpYWxpemF0aW9u...` (base64 >200 симв.) |
| **Почему секрет** | Сериализованный объект с высокой энтропией. Риск Insecure Deserialization. |
| **Как хранить** | Заменить на JWT с подписью. Для тестов - mock-объекты. Валидировать при десериализации. |

### 4 Password Hash
[пример](./image4.png)
| Параметр | Значение |
|----------|----------|
| **Расположение** | `.../MissingFunctionACUsersTest.java`, строка с `getUserHash()` |
| **Фрагмент** | `"cplTjehjI/e5ajqTxWaXhU5NW9UotJfXj+gcbPvfWwc="` |
| **Почему секрет** | Хеш пароля в коде. Возможен offline brute-force / rainbow table attack. |
| **Как хранить** | В тестах - mock. В продакшене - хеши с солью в защищённой БД. |

### 5 Plaintext Credentials
[пример](./image4.png)
| Параметр | Значение |
|----------|----------|
| **Расположение** | `.../MissingFunctionACUsersTest.java`, создание `WebGoatUser` |
| **Фрагмент** | `new WebGoatUser("user1","password1")`, `{"password":"newUser12"}` |
| **Почему секрет** | Пароли в явном виде. Нарушение password policy, риск компрометации аккаунтов. |
| **Как хранить** | Переменные окружения или внешние конфиги (.gitignore).  |

---

## Дополнительные вопросы

### 1. BFG Repo-Cleaner
Утилита для очистки истории Git от секретов и больших файлов.  
Удаляет данные из всех коммитов, а не только из HEAD (в отличие от git rm).  
**Пример:**
```bash
java -jar bfg.jar --delete-files '*.key' my-repo.git
git reflog expire --expire=now --all && git gc --prune=now
git push --force 
```
#### Способы автоматизации поиска секретов:
1. Pre-commit: gitleaks, truffleHog в .git/hooks/pre-commit
2. CI/CD: Скан в pipeline (GitHub Actions, GitLab CI) с allow_failure: false
3. Платформы: GitHub Advanced Security, GitLab Secret Detection, Snyk
4. Scheduled: Ночное сканирование всех репозиториев и алерты в Slack
