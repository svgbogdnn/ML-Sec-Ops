# ЛР 6. Поиск секретов в репозитории

**Репозиторий:** `https://github.com/OtusTeam/DevSecOps_secret-finding.git`  
**Инструмент:** `trufflehog`

---

## 1. GitHub token

**Расположение:** `.github/workflows/release.yml`, строка 72

GITHUB_TOKEN: cee3257f0e59d1d4975a35429cffc3d8b5fabe15ca2e9d47243cbc446d4894a5

**Почему это секрет:**  
Это токен GitHub. С его помощью можно выполнять действия в GitHub Actions и работать с репозиторием или артефактами сборки.

**Как правильно хранить:**  
Хранить в GitHub Actions Secrets и использовать так: GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

---

## 2. Приватный ключ

**Расположение:** `webgoat-server/privatekey.key`, строка 1


-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASC...
-----END PRIVATE KEY-----


**Почему это секрет:**  
Это приватный криптографический ключ. Его нельзя хранить в репозитории, потому что с его помощью можно подписывать данные, расшифровывать информацию и тд.

**Как правильно хранить:**  
В Secret Manager

---

## 3. CSR-файл

**Расположение:** `webgoat-server/certificatesigningrequest.csr`, строка 1

-----BEGIN CERTIFICATE REQUEST-----
MIIC4zCCAcsCAQAwdDELMAkGA1UEBhMCNDQ...
-----END CERTIFICATE REQUEST-----

**Почему это секрет / чувствительные данные:**  
CSR содержит данные для выпуска сертификата: публичный ключ и информацию о субъекте. Сам по себе CSR не является приватным ключом, но вместе с найденным приватным ключом относится к чувствительным криптографическим данным.

**Как правильно хранить:**  
Не хранить в публичном Git-репозитории. Генерировать при выпуске сертификата или хранить во внутреннем защищённом хранилище.

---

## 4. Пароль к PostgreSQL в connection string

**Расположение:** `docker-compose-postgres.yml`

jdbc:postgresql://webgoat_db:5432/webgoat?user=webgoat&password=webgoat

**Почему это секрет:**  
В строке подключения прямо указаны логин и пароль от базы данных PostgreSQL.

**Как правильно хранить:**  
Адрес базы можно оставить в конфиге, а логин и пароль нужно передавать через переменные окружения или Secret Manager.

---

## 5. Захардкоженные пароли и JWT secret

**Расположение:** `webgoat-lessons/challenge/src/main/java/org/owasp/webgoat/plugin/SolutionConstants.java`

String PASSWORD_TOM = "thisisasecretfortomonly";
String PASSWORD_LARRY = "larryknows";
String JWT_PASSWORD = "victory";
String ADMIN_PASSWORD_LINK = "375afe1104f4a487a73823c50a9292a2";

**Почему это секрет:**  
В коде хранятся пароли пользователей и секрет для JWT. Если злоумышленник узнает JWT secret, он может попытаться подделать токен.

**Как правильно хранить:**  
Пароли пользователей хранить только в виде хэшей, а JWT secret — в переменных окружения или Secret Manager.

---

## BFG Repo-Cleaner

BFG Repo-Cleaner — это инструмент для очистки истории Git. Он нужен, если секрет уже был закоммичен. Простого удаления строки из текущей версии файла недостаточно, потому что секрет остаётся в старых коммитах.

---

В ходе работы были найдены 5 типов секретов:

1. GitHub token.
2. Приватный ключ.
3. CSR-файл.
4. Пароль к PostgreSQL в строке подключения.
5. Захардкоженные пароли и JWT secret.

Для устранения проблем нужно удалить секреты из репозитория, перевыпустить скомпрометированные значения, очистить историю Git и хранить секреты только в защищённых хранилищах: GitHub Secrets, GitLab CI/CD Variables, Vault, Kubernetes Secrets или аналогичных решениях.
