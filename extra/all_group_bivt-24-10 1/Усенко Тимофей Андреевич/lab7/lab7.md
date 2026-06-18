# SAST-анализ мобильного приложения Telegram через MobSF

В рамках данной работы выполнено развертывание инструмента статического анализа 
MobSF (Mobile Security Framework) и проведено тестирование безопасности 
мобильного приложения Telegram.

- **Объект исследования:** `Telegram_11.1.1_APKPure.apk`
- **Идентификатор пакета:** `org.telegram.messenger`
- **Версия приложения:** 11.1.1
- **Размер:** 127.31 МБ
- **MD5:** b61762a4dc54b893a0ec79ada417920e
- **SHA256:** eba26231f8ddc306da94a169ed5cd67848f119d4cd6c7497945d3408a358d21d
- **Источник дистрибутива:** APKPure
- **Среда развертывания:** MobSF v4.5.0 в Docker-контейнере (локально, macOS)

---

## 1. Ход работы

Инструмент MobSF был запущен через Docker командой:

```bash
docker run -it --rm -p 8000:8000 --memory=6g \
  opensecurity/mobile-security-framework-mobsf:latest
```

После запуска контейнера открыт веб-интерфейс по адресу http://localhost:8000.
APK-файл загружен через кнопку Upload & Analyze. MobSF автоматически 
распаковал архив, декомпилировал код с помощью JADX, выполнил анализ манифеста, 
кода, разрешений и нативных библиотек, после чего сформировал итоговый отчёт.

---

## 2. Общие результаты сканирования

- **Security Score:** 50/100
- **Trackers Detection:** 0/432 (трекеры не обнаружены)
- **Target SDK:** 34, **Min SDK:** 23, **Max SDK:** —
- **Android Version Name:** 11.1.1
- **Подпись APK:** v1=True, v2=True, v3=True, v4=False

Распределение уязвимостей по разделам:

| Раздел | HIGH | WARNING | INFO |
|---|---|---|---|
| Certificate Analysis | 0 | 2 | 1 |
| Manifest Analysis | 2 | 36 | 0 |
| Code Analysis | 2 | 8 | 3 |

Сертификат подписи: издатель CN=Nikolay Kudashov, O=VK, L=Saint-Petersburg, 
алгоритм RSA, размер ключа 1024 бит, hash SHA1.

---

## 3. App Permissions — опасные разрешения (Dangerous)

| Разрешение | Описание |
|---|---|
| ACCESS_BACKGROUND_LOCATION | Доступ к геолокации в фоне |
| ACCESS_COARSE_LOCATION | Грубая геолокация (по сети) |
| ACCESS_FINE_LOCATION | Точная геолокация (GPS) |
| ACCESS_MEDIA_LOCATION | Доступ к геоданным в медиафайлах |
| AUTHENTICATE_ACCOUNTS | Выступать как аутентификатор аккаунтов |
| BLUETOOTH_CONNECT | Подключение к Bluetooth-устройствам |
| CALL_PHONE | Прямой вызов телефонных номеров |

Всего обнаружено 71 разрешение, из них около 30 имеют статус **dangerous**.

---

## 4. Certificate Analysis

| Проблема | Severity | Описание |
|---|---|---|
| Application vulnerable to Janus Vulnerability | WARNING | Приложение подписано схемой v1, что делает его уязвимым к Janus на Android 5.0–8.0 |
| Certificate algorithm might be vulnerable to hash collision | WARNING | Используется SHA1withRSA, известный коллизиями. Манифест указывает SHA256withRSA |
| Signed Application | INFO | Приложение подписано сертификатом разработчика |

---

## 5. Manifest Analysis (HIGH)

| № | Проблема | Severity | Описание |
|---|---|---|---|
| 1 | App can be installed on a vulnerable unpatched Android version (minSdk=23) | HIGH | Приложение можно установить на Android 6.0–6.0.1, в которых множество неустранённых уязвимостей |
| 2 | Clear text traffic is enabled (usesCleartextTraffic=true) | HIGH | Разрешён незашифрованный HTTP-трафик, что позволяет перехватывать и модифицировать данные |

Кроме того, обнаружено 36 WARNING, среди которых: 
- Application Data can be Backed up (`allowBackup=true`)
- Незащищённые экспортируемые компоненты: Service GcmPushListenerService, 
  Service GoogleVoiceClientService, Activity GoogleVoiceClientActivity, 
  Activity-Alias DefaultIcon, Activity-Alias VintageIcon и др.

---

## 6. Code Analysis (HIGH)

| № | Проблема | Severity | CWE / Стандарт |
|---|---|---|---|
| 1 | Remote WebView debugging is enabled | HIGH | CWE-919 (Weaknesses in Mobile Applications) / OWASP M1 / MSTG-RESILIENCE-2. Файл: `org/telegram/messenger/SharedConfig.java` |
| 2 | The App uses encryption mode CBC with PKCS5/PKCS7 padding — уязвим к padding oracle | HIGH | CWE-649 / OWASP M5 / MSTG-CRYPTO-3. Файл: `org/telegram/ui/bots/BotBiometry.java` |

Также обнаружено 8 WARNING, среди которых: 
- Files may contain hardcoded sensitive information (CWE-312)
- App uses insecure Random Number Generator (CWE-330)
- SHA-1 is a weak hash known to have hash collisions (CWE-327)
- App can read/write to External Storage (CWE-276)
- Insecure WebView Implementation (CWE-749)

---

## 7. Network Security

В разделе Network Security не зафиксировано отдельных правил с HIGH-уровнем — 
конфигурация network security не задана в манифесте. Однако основная проблема 
сетевой безопасности отражена в манифесте: разрешён cleartext-трафик 
(`usesCleartextTraffic=true`), что соответствует HIGH-уровню риска.

---

## 8. Подробный разбор ключевых уязвимостей

### Уязвимость №1 — Cleartext Traffic Enabled
- **Severity:** HIGH
- **CWE:** CWE-319 (Cleartext Transmission of Sensitive Information)
- **Описание:** В манифесте установлен `android:usesCleartextTraffic="true"`. 
Это позволяет приложению отправлять и принимать данные по HTTP в открытом виде. 
Сетевой атакующий (Wi-Fi, MITM) может перехватывать и модифицировать трафик.
- **Митигация:** Установить `android:usesCleartextTraffic="false"` или внедрить 
Network Security Config с запретом cleartext-трафика. Все запросы перевести на HTTPS.

### Уязвимость №2 — Установка на устаревший Android (minSdk=23)
- **Severity:** HIGH
- **Описание:** Приложение можно установить на Android 6.0, в котором есть 
непатченные уязвимости. Google рекомендует minSdk не ниже 29.
- **Митигация:** Поднять `minSdkVersion` до 29 и выше.

### Уязвимость №3 — Remote WebView Debugging Enabled
- **Severity:** HIGH
- **CWE:** CWE-919 (Weaknesses in Mobile Applications)
- **Файл:** `org/telegram/messenger/SharedConfig.java`
- **Описание:** Включена удалённая отладка WebView (`setWebContentsDebuggingEnabled(true)`). 
В production-сборке это позволяет атакующему через ADB подключиться к WebView 
и просматривать/модифицировать контент.
- **Митигация:** Отключать отладку WebView в release-сборках, например через 
`if (BuildConfig.DEBUG) WebView.setWebContentsDebuggingEnabled(true);`.

### Уязвимость №4 — Использование AES/CBC с PKCS5/PKCS7 (Padding Oracle)
- **Severity:** HIGH
- **CWE:** CWE-649 (Reliance on Obfuscation or Encryption without Integrity Checking)
- **Файл:** `org/telegram/ui/bots/BotBiometry.java`
- **Описание:** Используется режим шифрования CBC с PKCS5/PKCS7 padding без 
контроля целостности. Такая конфигурация уязвима к padding oracle attacks.
- **Митигация:** Перейти на AES/GCM (аутентифицированное шифрование), либо 
добавлять HMAC поверх CBC для контроля целостности.

### Уязвимость №5 — Janus Vulnerability (подпись только v1)
- **Severity:** WARNING (HIGH в зависимости от целевой платформы)
- **Описание:** Подпись v1 без v2/v3 на Android 5.0–7.0 позволяет внедрить 
вредоносный код в DEX так, что подпись остаётся валидной. В нашем случае 
подписи v2 и v3 присутствуют — риск частично снижен, но устаревшая схема v1 
остаётся в наборе.
- **Митигация:** Отключить подпись v1, использовать только v2/v3/v4.

### Уязвимость №6 — Избыточные опасные разрешения
- **Severity:** HIGH (в совокупности)
- **CWE:** CWE-250 (Execution with Unnecessary Privileges)
- **Описание:** Приложение запрашивает ACCESS_BACKGROUND_LOCATION, 
ACCESS_FINE_LOCATION, CALL_PHONE и др. — широкий набор опасных разрешений.
- **Митигация:** Запрашивать разрешения только в момент использования соответствующей 
функции (runtime permissions). Удалить разрешения, не используемые в текущей версии.

---

## 9. План митигации рисков

### 1. Сетевая безопасность
- Установить `android:usesCleartextTraffic="false"` в манифесте
- Внедрить Network Security Configuration с запретом cleartext-трафика
- Включить Certificate Pinning для защиты от MITM-атак

### 2. Криптография
- Заменить AES/CBC + PKCS5/PKCS7 на AES/GCM в `BotBiometry.java`
- Заменить SHA-1 на SHA-256 в `Utilities.java` и `PassportActivity.java`
- Использовать криптографически стойкий `SecureRandom` вместо обычного `Random`

### 3. Подпись приложения
- Перейти с RSA-1024 на RSA-2048 (или ECDSA)
- Отключить подпись v1, оставить только v2/v3/v4
- Использовать только SHA256withRSA для подписи

### 4. WebView
- Отключить `setWebContentsDebuggingEnabled` в release-сборках
- Валидировать загружаемые URL по белому списку
- Ограничить использование JavaScript в WebView

### 5. Манифест и компоненты
- Поднять `minSdkVersion` до 29
- Установить `android:allowBackup="false"` для защиты данных
- Закрыть экспорт незащищённых сервисов и активити (`android:exported="false"`)

### 6. Разрешения
- Провести аудит запрашиваемых разрешений
- Удалить неиспользуемые опасные разрешения
- Все опасные разрешения запрашивать в runtime

### 7. Проверка результатов
- После внесения исправлений повторно запустить сканирование в MobSF
- Контролировать рост Security Score (цель: 75+/100)

---

## 10. Вывод

В ходе работы был развёрнут инструмент MobSF в Docker-контейнере и проведён 
статический анализ мобильного приложения Telegram версии 11.1.1. По итогам 
сканирования итоговый Security Score составил 50/100, что указывает на наличие 
существенных проблем безопасности.

Среди критических уязвимостей: разрешённый cleartext-трафик, включённая 
удалённая отладка WebView, использование уязвимого режима шифрования AES/CBC 
с PKCS-padding, возможность установки на устаревшие версии Android и большое 
количество избыточных опасных разрешений.

Предложенный план митигации позволит устранить выявленные недостатки и 
повысить уровень защищённости приложения. После внесения исправлений 
рекомендуется провести повторное сканирование для контроля результата.

