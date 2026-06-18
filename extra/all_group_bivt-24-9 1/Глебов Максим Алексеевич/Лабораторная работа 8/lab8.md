# Лабораторная работа 8

**Тема:** SAST-анализ мобильного приложения с помощью MobSF и план митигации рисков.

---

# Цель

Развернуть инструмент SAST-анализа мобильных приложений (MobSF), просканировать мобильное приложение и составить план по митигации найденных рисков.

---

# Развёртывание MobSF

Развернул MobSF в Docker:

```bash
docker pull opensecurity/mobile-security-framework-mobsf:latest
docker run -d --name mobsf -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
docker logs mobsf | tail -20
```

В логах получил версию и REST API Key:

```text
Mobile Security Framework v4.5.0
Running on http://0.0.0.0:8000
REST API Key: 79ddd0f8...
Default Credentials: mobsf/mobsf
```

Веб-интерфейс доступен на http://localhost:8000 (логин `mobsf/mobsf`). Сканирование запускал через REST API: загрузка APK (`/api/v1/upload`), скан (`/api/v1/scan`), выгрузка отчёта в PDF (`/api/v1/download_pdf`).

---

# Объект тестирования

Для теста взял заведомо уязвимое приложение **InsecureBankv2** (учебное банковское приложение для отработки мобильной безопасности).

| Параметр | Значение |
|---|---|
| Приложение | InsecureBankv2 |
| Package | `com.android.insecurebankv2` |
| Версия | 1.0 |
| MD5 | `5ee4829065640f9c936ac861d1650ffc` |
| SHA256 | `b18af2a0e44d7634bbcdf93664d9c78a2695e050393fcfbb5e8b91f902d194a4` |
| min SDK / target SDK | 15 / 22 |
| **Security Score (MobSF)** | **28 / 100** |
| Итог сканирования | 7 проблем High, 9 Warning |

Полный отчёт MobSF приложен: `MobSF_InsecureBankv2_report.pdf`.

---

# Основные проблемы по результатам сканирования

## 1. App permissions со статусом dangerous

MobSF отметил 7 опасных (dangerous) разрешений:

| Разрешение | Что даёт |
|---|---|
| `WRITE_EXTERNAL_STORAGE` | чтение/изменение/удаление файлов на внешнем хранилище |
| `SEND_SMS` | отправка SMS (может использоваться для платных действий) |
| `USE_CREDENTIALS` | использование учётных данных аккаунта |
| `GET_ACCOUNTS` | доступ к списку аккаунтов на устройстве |
| `READ_PROFILE` | чтение персонального профиля пользователя |
| `READ_CONTACTS` | чтение контактов |
| `ACCESS_COARSE_LOCATION` | определение местоположения по сети |

**Риск:** приложение запрашивает доступ к SMS, контактам, аккаунтам и хранилищу. Для банковского приложения такой набор избыточен и расширяет поверхность атаки.

## 2. Network security (severity high и выше)

В этом сканировании MobSF **не выявил** отдельных network security находок уровня high+ (раздел Network Security пуст). Причина: у приложения нет файла Network Security Config, а target SDK = 22, где cleartext-трафик (HTTP) разрешён по умолчанию. Это само по себе риск, поэтому в план митигации я включил добавление Network Security Config.

## 3. Manifest analysis (severity high и выше)

High-находки манифеста:

| Находка | Описание |
|---|---|
| `android:debuggable=true` | включён debug, приложение легко подцепить отладчиком и снять дамп |
| `minSdk=15` | можно установить на Android 4.0.3-4.0.4 с непатченными уязвимостями |
| StrandHogg 2.0 | активности `PostLogin`, `DoTransfer`, `ViewStatement`, `ChangePassword` уязвимы к перехвату задач (task hijacking) |

Warning-находки манифеста (тоже важные):

- `android:allowBackup=true` - данные приложения можно выгрузить через `adb backup`;
- экспортированы без защиты: 4 активности (`PostLogin`, `DoTransfer`, `ViewStatement`, `ChangePassword`), Content Provider `TrackUserContentProvider`, Broadcast Receiver `MyBroadCastReceiver`. Их может вызвать любое другое приложение на устройстве.

## 4. Code analysis (severity high и выше)

High-находки уровня приложения и кода:

| Находка | Описание |
|---|---|
| Janus (v1 signature) | приложение подписано только схемой v1, уязвимо к Janus на Android 5.0-8.0 (подмена кода без нарушения подписи) |
| Hardcoded secrets | MobSF нашёл захардкоженные строки, похожие на секреты (в т.ч. base64-зашифрованные значения и связку login/password) |
| Privacy trackers | в приложении 3 трекера: Google AdMob, Google Analytics, Google Tag Manager |

Пример найденных захардкоженных секретов:

```text
"loginscreen_username" : "Username:"
"loginscreen_password" : "Password:"
qfDkyRZiTZGguvBzojuWMEqfI8Qqw5CcMB2eo7wr2iH9X2v+qlFOYNd9v9ffS1x0
EwZMQOzAsSbCW+73vnMc0IIAOIXmhdEPDWA4pBmTQFs=
```

---

# План по митигации рисков

| # | Недостаток | Риск | Митигация |
|---|---|---|---|
| 1 | Опасные разрешения (SMS, контакты, аккаунты, хранилище) | избыточный доступ, расширение поверхности атаки | оставить только реально нужные разрешения; перейти на scoped storage; SMS/контакты убрать, если не используются |
| 2 | Нет Network Security Config, target SDK 22 | разрешён cleartext HTTP, риск MITM | добавить `network_security_config.xml` с `cleartextTrafficPermitted=false`, включить TLS и certificate pinning, поднять target SDK |
| 3 | `android:debuggable=true` | лёгкая отладка и дамп памяти в проде | убрать debuggable, в релизной сборке должно быть `false` |
| 4 | `minSdk=15` | установка на старые уязвимые версии Android | поднять `minSdkVersion` минимум до актуально поддерживаемой (например 24+) |
| 5 | StrandHogg 2.0 у активностей | перехват задач, фишинг поверх приложения | задать `taskAffinity=""` и `launchMode`, обновить target SDK, проверять состояние задач |
| 6 | `android:allowBackup=true` | выгрузка данных через adb backup | поставить `android:allowBackup="false"` |
| 7 | Экспортированные компоненты без защиты | вызов активностей/провайдера/ресивера чужими приложениями | `android:exported="false"` где не нужен внешний доступ; иначе защитить permission с уровнем `signature` |
| 8 | Janus (только v1 подпись) | подмена кода без нарушения подписи | подписать схемами v2/v3 (APK Signature Scheme v2+) |
| 9 | Захардкоженные секреты | утечка ключей/учёток при реверсе APK | убрать секреты из кода, хранить в Android Keystore / на сервере, не в ресурсах |
| 10 | Слабое шифрование и хранение | секреты лежат как base64/слабый шифр | использовать стойкие алгоритмы (AES-GCM), ключи в Keystore, не хранить чувствительные данные локально |
| 11 | Privacy trackers | передача данных третьим сторонам | пересмотреть необходимость трекеров, уведомить пользователя, дать opt-out |

---

# Итог

Развернул MobSF v4.5.0 в Docker и провёл статический анализ приложения InsecureBankv2. Security Score - 28 из 100, найдено 7 проблем уровня High и 9 Warning.

Основные недостатки: опасные разрешения, включённый debug, низкий minSdk, уязвимость StrandHogg 2.0 у ключевых активностей, разрешённый backup, незащищённые экспортированные компоненты, подпись только v1 (Janus) и захардкоженные секреты.

По каждому недостатку составлен план митигации (таблица выше). Главные шаги: убрать debug и backup, поднять minSdk/targetSdk, закрыть экспортированные компоненты, перейти на подпись v2/v3, добавить Network Security Config и убрать секреты из кода в Android Keystore.

Полный отчёт MobSF приложен в файле `MobSF_InsecureBankv2_report.pdf`.
