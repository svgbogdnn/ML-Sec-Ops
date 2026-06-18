В этой работе я запускал MobSF через докер образ и прогонял через него APK `com.vkontakte.android_53441_rs.apk`

Приложение: VK 8.180  
Package Name: `com.vkontakte.android`

Результат общего сканирования такой:

Security Score: 49/100

По scorecard MobSF показал:

- HIGH: 3
- MEDIUM: 100
- INFO: 2

Также MobSF отметил, что в приложении найдено 13 privacy trackers.

---

# High severity

## Dangerous permissions

MobSF отметил несколько разрешений со статусом dangerous:

1. `android.permission.RECORD_AUDIO`
2. `android.permission.CAMERA`
3. `android.permission.ACCESS_COARSE_LOCATION`
4. `android.permission.ACCESS_FINE_LOCATION`
5. `android.permission.READ_CONTACTS`
6. `android.permission.WRITE_CONTACTS`
7. `android.permission.READ_CALENDAR`
8. `android.permission.WRITE_CALENDAR`
9. `android.permission.READ_PHONE_STATE`
10. `android.permission.CALL_PHONE`
11. `android.permission.GET_ACCOUNTS`
12. `android.permission.READ_EXTERNAL_STORAGE`
13. `android.permission.WRITE_EXTERNAL_STORAGE`

Такие разрешения дают приложению доступ к чувствительным данным пользователя: камере, микрофону, геолокации, контактам, календарю, аккаунтам, звонкам и файлам.

Для VK часть этих разрешений может быть ожидаемой, потому что приложение поддерживает звонки, фото, видео, сообщения и работу с контактами. Но всё равно каждое dangerous permission нужно отдельно обосновывать и запрашивать только тогда, когда оно реально нужно.

То есть первый обязательный пункт задания выполняется: dangerous permissions у приложения есть.

---

## Network Security

MobSF нашёл проблему уровня High:

`Domain config is insecurely configured to permit clear text traffic to these domains in scope.`

Cleartext traffic разрешён для:

- `127.0.0.1`
- `localhost`
- `mobileid.megafon.ru`
- `idgw.mobileid.mts.ru`
- `he-mc.tele2.ru`
- `he-mc.t2.ru`
- `beeline.ru`

Это означает, что для части доменов приложение допускает незашифрованный HTTP-трафик. Такой трафик потенциально можно перехватить.

Из всех находок это одна из самых важных, потому что сетевой трафик должен передаваться только по HTTPS.

---

## Manifest Analysis

В разделе Manifest Analysis high findings нет.

Но MobSF показал много warning, например:

1. `Application Data can be Backed up [android:allowBackup=true]`
2. Много Activity, Service и Broadcast Receiver с `android:exported=true`
3. Использование `taskAffinity`

---

## Code Analysis

В разделе Code Analysis для этого APK явные high findings не были подробно представлены.

При этом MobSF показал признаки защитных механизмов:

- Anti-VM Code
- Anti-Debug Code
- Anti-Disassembly Code

Также были проблемы с декомпиляцией через JADX.

Это не обязательно уязвимость, но это значит, что автоматический анализ кода мог быть неполным. Поэтому код лучше дополнительно проверить вручную.

---

# Дополнительно

## 1) Слабый алгоритм подписи сертификата

MobSF отметил:

`Certificate algorithm vulnerable to hash collision`

Причина в том, что приложение подписано с использованием `SHA1withRSA`.

SHA-1 давно считается устаревшим алгоритмом, поэтому это нормальное high-срабатывание, на которое стоит обратить внимание.

## 2) Privacy trackers

MobSF отметил 13 privacy trackers.

Сами по себе трекеры не всегда являются уязвимостью, но это риск для приватности, потому что приложение может собирать и передавать данные пользователя сторонним SDK.

---

# План митигации

Теперь план митигации.

### 1) Убрать cleartext traffic

Что делать:

- запретить cleartext traffic в production-сборке
- проверить `network_security_config`
- оставить только HTTPS
- удалить тестовые HTTP-исключения, если они не нужны

### 2) Пересмотреть dangerous permissions

Что делать:

- проверить, действительно ли нужны камера, микрофон, геолокация, контакты, календарь и звонки
- удалить лишние permissions из `AndroidManifest.xml`
- запрашивать разрешения только в момент использования функции
- объяснять пользователю, зачем нужен доступ

### 3) Исправить настройки Manifest

Что делать:

- отключить `android:allowBackup`, если резервное копирование не требуется
- закрыть лишние exported-компоненты через `android:exported=false`
- для реально внешних компонентов добавить permission-защиту
- проверить использование `taskAffinity`

### 4) Обновить схему подписи

Что делать:

- не использовать SHA-1
- перейти на более современный алгоритм, например `SHA256withRSA`
- использовать ключ не менее 2048 бит
- проверить процесс релизной сборки

### 5) Провести дополнительный анализ кода

Что делать:

- проверить код на hardcoded secrets
- проверить криптографию
- проверить WebView
- проверить логирование чувствительных данных
- при необходимости повторить анализ другим инструментом

### 6) Проверить сторонние SDK и трекеры

Что делать:

- посмотреть, какие SDK реально нужны
- удалить неиспользуемые SDK
- обновить используемые зависимости
- проверить, какие данные собирают privacy trackers
