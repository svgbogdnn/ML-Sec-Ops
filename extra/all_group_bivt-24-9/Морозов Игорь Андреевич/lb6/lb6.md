# ЛР 6. Поиск секретов в репозитории через truffleHog

## 1. Подготовка

По методичке надо поставить truffleHog через pip или в docker-контейнер `python:buster` и просканировать им учебный репозиторий

Попробовал в лоб:
```bash
pip3 install truffleHog
```

```text
error: externally-managed-environment
× This environment is externally managed
```

Это PEP 668 на свежем питоне. Через docker тоже не вышло, он не был запущен:
```bash
$ docker ps
failed to connect to the docker API at unix:///Users/fatuum/.docker/run/docker.sock
```

Не стал поднимать docker desktop ради одного скана, прошёлся по репозиторию руками. У truffleHog логика простая: regex + энтропия Шеннона по diff всех веток. Это можно повторить связкой `git log --all -p -G '<regex>'` и обычным grep


## 2. Осмотр веток

```bash
git clone https://github.com/OtusTeam/DevSecOps_secret-finding.git
cd DevSecOps_secret-finding
git branch -a
```

```text
* main
  remotes/origin/HEAD -> origin/main
  remotes/origin/develop
  remotes/origin/main
```

На `main` лежит только пустой `README.md`. Весь код WebGoat и подброшенные секреты на `develop`. Именно поэтому и надо сканировать все ветки

```bash
git checkout develop
git log --oneline | head -5
```

```text
14becbb2 Update pom.xml
1e4e7111 Update README.MD
34f1faad your commit
39148828 Initial commit
5107e111 test url fix
```

Коммит `34f1faad your commit` подозрительный уже по сообщению. Посмотрел подробнее:

```bash
git show --stat 34f1faad
```

```text
commit 34f1faad298b13e515a62330f593dac142506789
Author: Lookatshow <lookatshow.a@yandex.ru>
Date:   Mon Nov 22 23:20:34 2021 +0700
    your commit

 .github/workflows/release.yml                |  2 +-
 docker/.env                                  |  4 +++
 docker/docker-compose.yml                    | 42 ++++++++++++++++++++++++++++
 webgoat-server/certificatesigningrequest.csr | 18 ++++++++++++
 webgoat-server/privatekey.key                | 28 +++++++++++++++++++
```

Все секреты лежат в этом одном коммите. Дальше по штукам


## 3. Найденные секреты

### 1. GitHub Token

`.github/workflows/release.yml`, строка 75. В коммите подмена `${{ secrets.GITHUB_TOKEN }}` на голую строку:

```diff
- GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
+ GITHUB_TOKEN: cee3257f0e59d1d4975a35429cffc3d8b5fabe15ca2e9d47243cbc446d4894a5
```

64 hex-символа, очень похоже на classic github PAT (в 2021 году префиксы `ghp_`, `gho_` ещё не были обязательными). С таким токеном можно публиковать релизы, а при широких scope - пушить в репо и читать приватные

Как надо: вернуть подстановку `${{ secrets.GITHUB_TOKEN }}` обратно. Этот токен github генерирует автоматически для каждого запуска и сам же удаляет после. Если нужен личный токен с расширенными правами, его надо хранить в настройках репозитория (settings -> actions -> secrets), а не писать в коде. Уже утёкший токен в любом случае надо отозвать в настройках github, иначе он останется рабочим даже после удаления из репозитория


### 2. Приватный RSA-ключ

`webgoat-server/privatekey.key`. Полный приватный ключ в репе:

```text
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCpuOH6AnMc/xdJ
... (вырезано) ...
oX44T7mclitCYaOuoRnC2V5H
-----END PRIVATE KEY-----
```

Заголовок без `RSA` это формат PKCS#8. Рядом лежит CSR на `Test/Test/Test`, значит пара явно генерировалась под выпуск сертификата. Если им подписан сертификат сервера, любой кто склонировал репо может расшифровать перехваченный трафик или прикинуться этим сервером

Что надо делать: генерировать ключ на сервере при старте, либо доставать из vault / KMS / sealed-secrets. `*.key`, `*.pem`, `id_rsa*` в глобальный `.gitignore`


### 3. `.env` с паролями

`docker/.env`:

```text
APP_ENV=dev
APP_SECRET=666666666666666
USER=alex675
PASSWORD=#!qerge34t423sdfvdsfgeR
```

Тут сразу два секрета. `APP_SECRET` это ключ, которым приложение подписывает данные о входе пользователя (то есть cookie сессии или JWT). По нему сервер потом понимает, что cookie не подделана. Если ключ известен атакующему, он может сгенерировать себе cookie от имени любого пользователя и зайти как он. В нашем случае ключ это пятнадцать шестёрок, то есть никакого секрета по сути и нет

`USER` и `PASSWORD` это логин и пароль от приватного репозитория пакетов composer. Дальше в dockerfile эти значения подставляются в команду авторизации перед установкой зависимостей. Если они утекли, атакующий получает доступ к корпоративным PHP пакетам

Как чинить:
- сгенерировать новый `APP_SECRET` командой `openssl rand -hex 32` (это даст 32 случайных байта в виде hex-строки) и положить его в переменные окружения сервера или в защищённое хранилище
- логин и пароль composer переложить в секреты CI (в настройках проекта), а не в коде
- из репозитория удалить сам `.env` и оставить только пример `.env.example`, где написаны имена переменных без значений, чтобы новый разработчик понимал, что надо заполнить


### 4. Echo секретов в Dockerfile

`docker/docker-compose.yml` (файл назван docker-compose, но это Dockerfile):

```dockerfile
RUN echo "${USER}"
RUN echo "${PASSWORD}"
...
RUN composer config http.libraries.com "${USER}" "${PASSWORD}" --global
```

Даже если `USER`/`PASSWORD` придут из CI секретов, `RUN echo "${PASSWORD}"` запишет пароль в build log и в слой образа. Любой, кто скачает образ, увидит пароль через `docker history`

Как чинить: убрать echo. Для composer авторизации использовать `--mount=type=secret` из buildkit, тогда секрет не остаётся в слоях


### 5. CSR

`webgoat-server/certificatesigningrequest.csr`. Сам CSR это публичная часть и формально не секрет, но trufflehog засветит его по высокой энтропии base64. Тут важно понять что это маркер: если рядом с CSR лежит `.key`, значит разработчик хранит здесь и приватную часть тоже. Хорошая зацепка для ручной проверки


## 4. Что ещё проверил по истории

Чтобы убедиться, что больше секретов нет нигде, прогнал отдельные greps

Приватные ключи по всему git:
```bash
git log -p --all -G 'BEGIN (RSA |EC |OPENSSH |)PRIVATE KEY' | grep -c "PRIVATE KEY"
```
Только пары из `34f1faad`

Длинные hex (потенциальные токены):
```bash
git log -p --all -G '[A-Fa-f0-9]{40,}'
```
Кроме `GITHUB_TOKEN: cee3257f...` нашлись только контрольные суммы (sha-хеши) зависимостей в файле `pom.xml`. Это просто способ проверить, что скачанная библиотека не подменена, и никакого секрета они не содержат. По факту это ложное срабатывание

По ключевым словам:
```bash
git grep -nE '(password|secret|token).*=' develop -- '*.properties' '*.env*'
```

```text
webgoat-container/src/main/resources/application-webgoat.properties:11:server.ssl.key-store-password=${WEBGOAT_KEYSTORE_PASSWORD:password}
```

Здесь используется синтаксис фреймворка spring: запись `${WEBGOAT_KEYSTORE_PASSWORD:password}` значит "взять пароль из переменной окружения `WEBGOAT_KEYSTORE_PASSWORD`, а если её нет, то использовать строку `password` как значение по умолчанию". То есть если на боевом сервере забыть выставить переменную, приложение запустится со слабым паролем `password`. На локальной машине для разработки это удобно, а на проде дыра в безопасности


## 5. Дополнительные вопросы

### BFG Repo-Cleaner

Утилита поверх `git filter-branch`. Умеет удалять файл по имени из всей истории, заменять строку на маркер и чистить большие бинарники. Под нашу задачу:

```bash
bfg --delete-files privatekey.key
bfg --replace-text passwords.txt
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push --force
```

Важно: BFG переписывает только историю, текущий HEAD надо чистить отдельно через `git rm`. И даже после чистки ротация ключей обязательна, репо могли уже склонировать

### Автоматизация поиска

Можно и нужно, в три слоя:
- проверка на машине разработчика перед коммитом (`gitleaks`, `detect-secrets`) - не пускает секрет даже до отправки в репозиторий
- проверка в CI на каждый запрос на слияние (`gitleaks detect`, `trufflehog filesystem`) - если что-то найдено, сборка падает и слияние блокируется
- сканер на стороне платформы (GitHub Advanced Security) ловит уже загруженные секреты по известным паттернам

Подводные камни: много ложных срабатываний на файлах с версиями зависимостей и тестах (нужен список исключений), локальную проверку можно обойти флагом `git commit --no-verify` (поэтому серверная проверка обязательна), сам сканер только фиксирует утечку - перевыпуск ключа всё равно делается руками


## Итого

Итого нашлось пять секретов, все из одного коммита `34f1faad` на ветке `develop`:

- github PAT в `.github/workflows/release.yml:75` - критично
- приватный RSA-ключ в `webgoat-server/privatekey.key` - критично
- `APP_SECRET` в `docker/.env:2` - высокая
- логин и пароль composer в `docker/.env:3-4` - высокая
- echo секретов в `docker/docker-compose.yml:27-31` - пойдет

На `main` этого коммита нет, и если сканировать только текущую рабочую копию, всё было бы пропущено. Сам truffleHog полезен, но без перевыпуска ключей и проверок до коммита он только констатирует факт уже случившегося
