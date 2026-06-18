## 1. Цель работы

Развернуть инструмент статического анализа мобильных приложений MobSF, провести SAST-тестирование реального Android-приложения и составить план митигации выявленных уязвимостей.


## 2. Развёртывание MobSF

Инструмент развёрнут через Docker:

```bash
docker pull opensecurity/mobile-security-framework-mobsf:latest

docker run -it --rm \
  -p 8000:8000 \
  -v $(pwd)/mobsf_data:/root/.MobSF \
  opensecurity/mobile-security-framework-mobsf:latest
```

Веб-интерфейс доступен по адресу `http://127.0.0.1:8000`. REST API ключ берётся из логов контейнера при первом запуске.

## 3. Выбор APK и проведение анализа

Для анализа выбран **Firefox for Android v131.0** — реальный браузер с широкими разрешениями и сетевой активностью. APK загружен через веб-интерфейс MobSF. PDF-отчёт приложен отдельным файлом `mobsf_firefox_report.pdf`.

**Общие результаты:**

```
App Security Score: 38/100
Grade: F
Risk Level: HIGH RISK

High:   6
Medium: 14
Info:   2
```

## 4. Опасные разрешения (статус: dangerous)

| Разрешение | Риск |
|------------|------|
| `ACCESS_FINE_LOCATION` | Точная GPS-геолокация, отслеживание перемещений |
| `ACCESS_COARSE_LOCATION` | Приблизительная геолокация по сети |
| `RECORD_AUDIO` | Запись звука с микрофона |
| `CAMERA` | Доступ к камере устройства |
| `READ_CONTACTS` | Чтение всей адресной книги |
| `READ/WRITE_EXTERNAL_STORAGE` | Полный доступ к файлам на накопителе |
| `SYSTEM_ALERT_WINDOW` | Наложение поверх других окон — вектор overlay-атак |

Разрешения на геолокацию, камеру и микрофон обоснованы функциональностью браузера (WebRTC, Geolocation API). `SYSTEM_ALERT_WINDOW` и `READ/WRITE_EXTERNAL_STORAGE` требуют пересмотра.

## 5. Network Security

| № | Проблема | Уровень |
|---|---------|---------|
| 1 | Base config разрешает cleartext-трафик | HIGH |
| 2 | Domain config разрешает HTTP для `tiles.services.mozilla.com` | HIGH |
| 3 | Конфигурация доверяет пользовательским CA | HIGH |
| 4 | Base config доверяет всем системным CA без ограничений | WARNING |

Разрешённый cleartext-трафик открывает возможность для MITM-атак в публичных Wi-Fi-сетях. Доверие к пользовательским CA позволяет злоумышленнику установить собственный сертификат и перехватить весь HTTPS-трафик приложения.

**Рекомендация — исправленный `network_security_config.xml`:**

```xml
<network-security-config>
  <base-config cleartextTrafficPermitted="false">
    <trust-anchors>
      <certificates src="system"/>
      <!-- user CA убраны из production -->
    </trust-anchors>
  </base-config>
  <debug-overrides>
    <trust-anchors>
      <certificates src="user"/>
    </trust-anchors>
  </debug-overrides>
</network-security-config>
```

## 6. Manifest Analysis (High и выше)

| № | Проблема | Уровень |
|---|---------|---------|
| 1 | `android:usesCleartextTraffic=true` | HIGH |
| 2 | `minSdkVersion=21` (Android 5.0, без патчей безопасности) | HIGH |

Дополнительно обнаружено 11 экспортированных компонентов без явной защиты permission (уровень WARNING): `MainActivity`, `ExternalAppBrowserActivity`, `MediaContentProvider` и другие.


## 7. Code Analysis (High и выше)

| № | Идентификатор | CWE | Файл | Описание |
|---|--------------|-----|------|----------|
| 1 | `android_world_readable` | CWE-276 | `geckoview/StorageController.java` | SharedPreferences создаётся в режиме `MODE_WORLD_READABLE` — любое приложение на устройстве может прочитать его содержимое |
| 2 | `android_insecure_random` | CWE-330 | `support/utils/CreditCardUtils.kt` | В криптографическом контексте используется `java.util.Random` вместо `SecureRandom` — значения предсказуемы |

Из WARNING: использование MD5 и SHA-1 в зависимостях, временные файлы в общих директориях, строки высокой энтропии (возможные захардкоженные ключи).


## 8. План митигации

| Приоритет | Проблема | Рекомендация |
|-----------|---------|-------------|
| **P0** | Cleartext HTTP + доверие user CA | `cleartextTrafficPermitted="false"` глобально; убрать `<certificates src="user"/>` из production |
| **P0** | `android:usesCleartextTraffic=true` | Удалить флаг; перевести все API-эндпоинты на HTTPS |
| **P1** | World Readable SharedPreferences (CWE-276) | Заменить `MODE_WORLD_READABLE` на `MODE_PRIVATE`; для чувствительных данных — `EncryptedSharedPreferences` |
| **P1** | `java.util.Random` в криптоконтексте (CWE-330) | Заменить на `java.security.SecureRandom` везде, где генерируются токены и ключи |
| **P2** | `minSdkVersion=21` | Поднять до `minSdkVersion=26` (Android 8.0) |
| **P2** | Экспортированные компоненты без permission | Добавить `android:permission` или явно установить `android:exported="false"` |
| **P3** | `SYSTEM_ALERT_WINDOW`, `READ/WRITE_EXTERNAL_STORAGE` | Удалить `SYSTEM_ALERT_WINDOW` если не нужно; перейти на Scoped Storage (Android 10+) |
| **P3** | MD5 и SHA-1 | Заменить на SHA-256; обновить зависимости |
| **P4** | Возможные захардкоженные секреты | Проверить строки вручную; реальные ключи вынести в Android Keystore |

## 9. Выводы

С помощью MobSF получен полный отчёт по APK Firefox for Android. Приложение набрало **38/100 (Grade F)**. Наиболее критичными находками являются разрешённый cleartext-трафик и доверие пользовательским CA (риск MITM), World Readable SharedPreferences (CWE-276) и небезопасный генератор случайных чисел в криптоконтексте (CWE-330).

MobSF удобен тем, что покрывает сразу несколько уровней анализа — разрешения, манифест, код, нативные бинари — и легко встраивается в CI/CD через REST API. Автоматическое сканирование каждой релизной сборки позволяет обнаруживать уязвимости до попадания в production, что соответствует принципу **Shift Left Security** в DevSecOps.

