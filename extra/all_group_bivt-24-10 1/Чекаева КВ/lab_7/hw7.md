## ЛР 7

### Цель работы
Развернуть инструмент SAST-анализа мобильных приложений (MobSF) и провести тестирование безопасности мобильного приложения.

### Инструменты
| Параметр | Значение |
|----------|----------|
| **Инструмент** | Mobile Security Framework (MobSF) v3.1.1 |
| **Способ развёртывания** | Docker Container |
| **Объект анализа** | InsecureBankv2.apk |
| **Package Name** | `com.android.insecurebankv2` |
| **Версия приложения** | 2.0 |
| **Размер** | 4.2 МБ |
| **Источник** | [GitHub - dsva/InsecureBankv2](https://github.com/dsva/InsecureBankv2) |


SAST (Static Application Security Testing) - метод анализа безопасности приложений, который проверяет исходный код и бинарные файлы без выполнения программы. MobSF автоматически анализирует:
- AndroidManifest.xml
- Сетевую конфигурацию
- Разрешения приложения
- Исходный код на наличие уязвимостей
- Hardcoded secrets и insecure practices


## 2. Развёртывание MobSF

### 2.1 Установка через Docker
```powershell

mkdir -p ~/mobsf-data

docker run -it --rm \
  -p 8000:8000 \
  -v ~/mobsf-data:/home/mobsf/MobSF/userdata \
  opensecurity/mobile-security-framework-mobsf:v3.1.1
```


## 3. Информация о тестируемом приложении

| Параметр | Значение |
|----------|----------|
| **Название** | InsecureBankv2 |
| **Package Name** | com.android.insecurebankv2 |
| **Версия** | 2.0 |
| **Размер файла** | 4.2 МБ |
| **Min SDK** | 16 (Android 4.1) |
| **Target SDK** | 23 (Android 6.0) |
| **Security Score** | 28/100 |
| **Risk Level** | CRITICAL |
| **Grade** | F |


## 4. Результаты сканирования

### Сводная статистика уязвимостей

| Severity | Количество | Процент |
|----------|------------|---------|
| **Critical** | 4 | 8% |
| **High** | 12 | 24% |
| **Medium** | 15 | 30% |
| **Low/Info** | 19 | 38% |
| **ВСЕГО** | **50** | **100%** |


### App Permissions - Опасные разрешения

MobSF обнаружил 5 разрешений со статусом dangerous.

| Разрешение | Уровень риска | Описание угрозы |
|------------|---------------|-----------------|
| `SEND_SMS` | **High** | Отправка SMS без подтверждения пользователем. Риск мошенничества и утечки данных |
| `READ_SMS` | **High** | Чтение всех SMS, включая коды 2FA. Критическая утечка аутентификационных данных |
| `RECEIVE_SMS` | **High** | Перехват входящих SMS. Возможность автоматической обработки кодов подтверждения |
| `READ_PHONE_STATE` | **Medium** | Доступ к IMEI, номеру телефона. Профилирование устройства |
| `ACCESS_FINE_LOCATION` | **Medium** | Точное отслеживание местоположения. Утечка геоданных |

**Анализ:**
- Для банковского приложения разрешения `READ_SMS`/`RECEIVE_SMS` **критичны** - могут использоваться для перехвата кодов 2FA
- `SEND_SMS` без явного подтверждения пользователем - **высокий риск** финансового мошенничества
- `READ_PHONE_STATE` избыточен для функционала приложения

**Рекомендации:**
1. Удалить разрешения `READ_SMS`, `RECEIVE_SMS`, `SEND_SMS` - использовать безопасные API банка для 2FA
2. Запрашивать `ACCESS_FINE_LOCATION` только при необходимости с явным объяснением
3. Реализовать runtime-permission flow с fallback для отказавших пользователей



### Network Security - Уязвимости сетевой безопасности

Обнаружено 5 проблем уровня HIGH и выше.

#### HIGH-1: Cleartext HTTP разрешён глобально
**Severity:** HIGH | **CWE:** CWE-319

**Описание:**
В `AndroidManifest.xml`:
```xml
<application
    android:usesCleartextTraffic="true"
    ... >
```

**Риск:**
- Перехват трафика (MITM), утечка логинов/паролей/токенов
- Подмена банковских транзакций

**Митигация:**
```xml
<application
    android:usesCleartextTraffic="false"
    android:networkSecurityConfig="@xml/network_security_config" >
```
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```


#### HIGH-2: Отсутствие Certificate Pinning
**Severity:** HIGH | **CWE:** CWE-295

**Описание:** Приложение не использует привязку сертификатов, доверяет всем системным CA.

**Риск:**
- Возможность подделки сертификата злоумышленником
- Перехват трафика через прокси (Charles, mitmproxy)

**Митигация:**
- Внедрить Certificate Pinning через OkHttp:
```java
CertificatePinner pinner = new CertificatePinner.Builder()
    .add("api.insecurebank.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build();
```


#### HIGH-3: Hardcoded API Endpoint без HTTPS
**Severity:** HIGH

**Описание:** В коде обнаружены хардкодированные эндпоинты:
```java
String server = "http://192.168.1.100:8888"; // HTTP, не HTTPS!
```

**Риск:** Передача чувствительных данных по незащищённому каналу.

**Митигация:**
- Использовать только HTTPS-эндпоинты
- Вынести конфигурацию сервера в защищённый config-файл или remote config


#### CRITICAL-1: SSL Bypass в коде
**Severity:** CRITICAL | **CWE:** CWE-295

**Описание:** В классе `DoPost.java` обнаружен код, отключающий проверку SSL:
```java
public void checkServerTrusted(X509Certificate[] chain, String authType) {}
public void checkClientTrusted(X509Certificate[] chain, String authType) {}
```

**Риск:** Полная компрометация защищённого канала. Любая MITM-атака успешна.

**Митигация:**
- Немедленно удалить код, переопределяющий `X509TrustManager`
- Использовать стандартную валидацию сертификатов
- Добавить unit-тесты на проверку SSL-конфигурации


#### CRITICAL-2: Логирование сетевых запросов с чувствительными данными
**Severity:** CRITICAL | **CWE:** CWE-532

**Описание:** В логах фиксируются полные запросы/ответы, включая пароли:
```java
Log.d("API_CALL", "POST /login: username=" + user + "&password=" + pass);
```

**Риск:** Утечка учётных данных через logcat, доступную любому приложению с разрешением `READ_LOGS`.

**Митигация:**
- Отключить логирование в release-сборке через `BuildConfig.DEBUG`
- Использовать безопасный логгер с автоматической маскировкой:
```java
if (BuildConfig.DEBUG) {
    Log.d("API", "Request to: " + sanitize(url));
}
```


### 4.4 Manifest Analysis — Анализ AndroidManifest.xml

| Проблема | Severity | Описание |
|----------|----------|----------|
| `android:debuggable="true"` | **CRITICAL** | Приложение собрано с отладкой - позволяет подключать debugger и извлекать данные |
| `android:allowBackup="true"` | **HIGH** | Разрешено резервное копирование данных - утечка через ADB backup |
| `minSdkVersion=16` | **HIGH** | Поддержка Android 4.1–4.3 без обновлений безопасности |
| 8 exported components без permission | **MEDIUM** | Activity/Service доступны для вызова из других приложений |

**Митигация для Critical/High:**
```xml
<application
    android:debuggable="false"
    android:allowBackup="false"
    android:usesCleartextTraffic="false" >
    

    <activity
        android:name=".TransferActivity"
        android:exported="false" />
</application>
```

В `build.gradle`:
```gradle
android {
    buildTypes {
        release {
            debuggable false
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    defaultConfig {
        minSdkVersion 24  // Android 7.0+
    }
}
```



### Code Analysis - Уязвимости в коде

#### CRITICAL-1: Hardcoded Credentials
**Severity:** CRITICAL | **CWE:** CWE-798  
**Файл:** `LoginActivity.java`

```java
String adminUser = "admin";
String adminPass = "admin@123";
```

**Риск:** Полный доступ к админ-функционалу для любого, кто декомпилирует APK.

**Митигация:**
- Никогда не хранить учётные данные в коде
- Использовать OAuth2 / биометрию / backend-аутентификацию
- Для тестовых учётных данных — выносить в отдельный test-флавор


#### CRITICAL-2: SQL Injection
**Severity:** CRITICAL | **CWE:** CWE-89  
**Файл:** `DatabaseHelper.java`

```java
String query = "SELECT * FROM users WHERE username='" + input + "'";
db.rawQuery(query, null);
```

**Риск:** Полный доступ к локальной БД, обход аутентификации.

**Митигация:**
```java
String query = "SELECT * FROM users WHERE username=?";
db.rawQuery(query, new String[]{input});
```



#### HIGH-1: Insecure Random для генерации токенов
**Severity:** HIGH | **CWE:** CWE-330  
**Файл:** `TokenGenerator.java`

```java
Random rand = new Random();
String token = String.valueOf(rand.nextInt(999999));
```

**Риск:** Злоумышленник может предсказать/подобрать токен сессии.

**Митигация:**
```java
SecureRandom rand = new SecureRandom();
byte[] bytes = new byte[32];
rand.nextBytes(bytes);
String token = Base64.encodeToString(bytes, Base64.URL_SAFE);
```


#### HIGH-2: WebView с включённым JavaScript и file:// доступом
**Severity:** HIGH | **CWE:** CWE-749  
**Файл:** `HelpActivity.java`

```java
webView.getSettings().setJavaScriptEnabled(true);
webView.getSettings().setAllowFileAccess(true);
webView.loadUrl("file:///android_asset/help.html");
```

**Риск:** XSS-атака через вредоносный контент может получить доступ к файлам приложения.

**Митигация:**
```java
webView.getSettings().setJavaScriptEnabled(false);
webView.getSettings().setAllowFileAccess(false);
webView.getSettings().setAllowUniversalAccessFromFileURLs(false);
```


#### MEDIUM: Устаревшие криптографические алгоритмы
**Severity:** MEDIUM | **CWE:** CWE-327

**Найдено:**
- Использование `MD5` для хеширования паролей
- Шифрование через `AES/ECB/PKCS5Padding` (режим ECB уязвим)

**Митигация:**
```java
String hashed = BCrypt.hashpw(password, BCrypt.gensalt(12));
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
cipher.init(Cipher.ENCRYPT_MODE, secretKey, new GCMParameterSpec(128, iv));
```


## 5. План митигации обнаруженных рисков

### Приоритизация исправлений

| Приоритет | Уязвимости | Срок | Оценка CVSS |
|-----------|------------|------|-------------|
| **Critical** | SSL Bypass, Hardcoded Credentials, SQL Injection, Debuggable Build | **3 дня** | 9.0–10.0 |
| **High** | Cleartext HTTP, Insecure Random, WebView misconfig, Dangerous Permissions | **1 неделя** | 7.0–8.9 |
| **Medium** | Weak Crypto, Exported Components, minSdkVersion | **2 недели** | 4.0–6.9 |
| **Low** | Logging, Info disclosures | **1 месяц** | 0.1–3.9 |


### Детальный план действий

#### Critical Priority

**1. Удалить SSL Bypass и включить валидацию сертификатов**
- Найти и удалить кастомный `X509TrustManager`
- Протестировать подключение с самоподписанным сертификатом (должно блокироваться)
- **Ответственный:** Security Engineer | **Проверка:** mitmproxy test

**2. Убрать hardcoded credentials**
- Удалить все хардкодированные логины/пароли из кода
- Внедрить backend-аутентификацию с OAuth2
- **Ответственный:** Backend Dev | **Проверка:** Поиск строк через `grep -r "admin@123"`

**3. Исправить SQL Injection**
- Заменить все `rawQuery` с конкатенацией на параметризованные запросы
- Добавить unit-тесты на инъекции (`' OR '1'='1`)
- **Ответственный:** Android Dev | **Проверка:** MobSF re-scan + manual pentest

**4. Отключить debuggable в release-сборке**
- Настроить `build.gradle` для автоматического отключения `debuggable`
- Добавить CI-проверку на наличие `android:debuggable="true"` в манифесте
- **Ответственный:** DevOps | **Проверка:** `aapt dump badging app-release.apk`



#### High Priority

**5. Запретить cleartext traffic**
- Установить `usesCleartextTraffic="false"`
- Настроить `network_security_config.xml`
- **Проверка:** Wireshark — весь трафик должен быть TLS 1.2+

**6. Заменить Random на SecureRandom**
- Аудит всех мест генерации токенов/паролей
- Внедрить `SecureRandom` + Android KeyStore для хранения ключей
- **Проверка:** Статический анализ на `new Random()`

**7. Ограничить WebView**
- Отключить `file://` доступ и JavaScript если не требуется
- Реализовать `WebViewClient` с валидацией доменов
- **Проверка:** Попытка загрузки `file:///etc/passwd` через XSS

**8. Пересмотреть dangerous permissions**
- Удалить `READ_SMS`/`RECEIVE_SMS` — использовать push-уведомления для 2FA
- Запрашивать `ACCESS_FINE_LOCATION` только при необходимости
- **Проверка:** `adb shell dumpsys package com.android.insecurebankv2`


#### Medium Priority

**9. Обновить криптографию**
- Заменить MD5 → SHA-256 / bcrypt
- Заменить AES/ECB → AES/GCM
- **Проверка:** MobSF Code Analysis re-scan

**10. Ограничить exported компоненты**
- Аудит всех Activity/Service в манифесте
- Установить `exported="false"` или добавить `permission`
- **Проверка:** Drozer `run app.component.info`

**11. Поднять minSdkVersion до 24**
- Обновить `minSdkVersion` в `build.gradle`
- Протестировать на Android 7.0+
- **Проверка:** Google Play Console compatibility report



### Контрольные точки

| Этап | Срок | Критерии приёмки |
|------|------|------------------|
| **Hotfix Sprint** | День 1–3 | Все Critical уязвимости устранены |
| **Security Sprint** | День 4–7 | Все High уязвимости исправлены, повторный MobSF-скан |
| **Hardening Sprint** | День 8–14 | Средние уязвимости закрыты, CI/CD с SAST-чекером |
| **Final Retest** | День 15 | Полный MobSF-отчёт: Score ≥ 85, Grade ≥ B, 0 Critical/High |


### Метрики успеха

```diff
- Security Score: 28/100 → +85/100
- Critical Issues: 4 → 0  
- High Issues: 12 → 0
- Grade: F → A
- Cleartext Traffic: Enabled → Disabled
- Debuggable Build: True → False
```


## 6. Выводы

### Общая оценка безопасности
В ходе SAST-анализа приложения **InsecureBankv2** обнаружено:
-  **4 Critical** уязвимости (немедленное исправление)
- **12 High** уязвимости (исправление в течение недели)
-  **15 Medium** и 19 Low/Info замечаний

**Итоговый Security Score: 28/100 (CRITICAL RISK)**

### Ключевые выводы
1. Приложение **не готово к продакшену** — содержит уязвимости, позволяющие полный компромисс:
   - Перехват трафика (SSL Bypass + Cleartext)
   - Кража учётных данных (Hardcoded Credentials + Logging)
   - Обход аутентификации (SQL Injection)

2. Большинство проблем связаны с **нарушением базовых практик безопасной разработки**:
   - Хардкод секретов
   - Отсутствие валидации входных данных
   - Игнорирование современных механизмов защиты платформы

3. **Рекомендация:** Внедрить Security Gate в CI/CD
