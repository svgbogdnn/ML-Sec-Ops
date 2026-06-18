# Лабораторная работа №7: SAST-анализ мобильного приложения (MobSF)

## Выполнение

Запустил MobSF в Docker:
docker run -d -p 8000:8000 --name mobsf opensecurity/mobile-security-framework-mobsf:v3.1.1

Для анализа взял приложение Simple Notes (com.simplemobiletools.notes). APK скачан с apkmirror.com, загружен в MobSF. Полный PDF-отчёт сохранён и приложен.

## Обнаруженные проблемы и план митигации

### 1. App Permissions (dangerous)
Найдены опасные разрешения: INTERNET, WRITE_EXTERNAL_STORAGE, com.android.vending.BILLING, BIND_APPHUB_SERVICE.

Риск: передача данных без ведома пользователя, доступ к файлам, сбор информации рекламными SDK.

План митигации:
- Оставить только нужные для заметок разрешения.
- Запрашивать доступ к файлам во время выполнения с объяснением.
- Удалить биллинг и рекламные сервисы, если не используются.

### 2. Network Security (high и выше)
В разделе Network Security уязвимостей нет, но в коде есть http-ссылки.

План митигации:
- Заменить http на https.
- Настроить Network Security Config с запретом открытого трафика.
- Явно указать доверенные сертификаты.

### 3. Manifest Analysis (high и выше)
Множество экспортируемых компонентов не защищены (20 Activity-Alias, MainActivity, сервисы и приёмники). allowBackup=true.

Риск: любое приложение может вызвать эти компоненты, резервное копирование раскроет заметки.

План митигации:
- Установить exported=false для непубличных компонентов.
- Для публичных (Widget) добавить permission с level=signature.
- Установить allowBackup=false.

### 4. Code Analysis (high и выше)
Критические находки (все high):
- Небезопасный Random (SecureRandom не используется).
- Жёстко закодированные ключи в коде AppLovin, Yandex Metrica.
- Небезопасный WebView (игнорирует ошибки SSL) — MITM-уязвимость.
- Слабые хеши SHA-1, MD5.
- Чтение/запись во внешнее хранилище.
- Включена удалённая отладка WebView.

План митигации:
- Заменить Random на SecureRandom.
- Убрать ключи из кода, хранить в Android Keystore.
- В WebView проверять SSL-сертификаты.
- SHA-1/MD5 заменить на SHA-256, пароли — bcrypt.
- Использовать getExternalFilesDir вместо открытой SD-карты.
- Отключить удалённую отладку WebView: setWebContentsDebuggingEnabled(false).
- Применить ProGuard/R8.

## Выводы

MobSF выявил проблемы даже в простом приложении: опасные разрешения, уязвимости в сторонних SDK, небезопасный WebView, слабую криптографию. План митигации устраняет эти риски. Инструмент можно внедрить в CI/CD.