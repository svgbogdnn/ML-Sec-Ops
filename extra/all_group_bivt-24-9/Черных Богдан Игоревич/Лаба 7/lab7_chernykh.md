# Лабораторная Работа №7

*Черных Богдан Игоревич ' БИВТ-24-9*

---

### Условие

Развернуть инструмент SAST-анализа и провести SAST тестирование
Цель:
Произвести установку инструмента SAST-анализа мобильных приложений (MobSF).
Провести SAST тестирование для любого мобильного приложения.
Составить план по митигации обнаруженных рисков.

Описание/Пошаговая инструкция выполнения домашнего задания:
Воспользуйтесь официальной документаций MobSF, выполните развертывание инструмента любым доступным вам способом (standalone под linux/windows, в docker, или в kubernetes). Используйте файл, приложенный к материалам вебинара.
Скачать любое доступные мобильное приложение (предпочтительно использовать .apk (android) или .ipa (iOS) файлы)
Проведите сканирование дистрибутива мобильного приложения
Выделите основные проблемы в результатах сканирования:
app permissions со статусом dangerous
network security с severity уровня high и выше
manifest analysis с severity уровня high и выше
code analysis с severity уровня high и выше
Выполните анализ по результатам проведенного сканирования, подсветите обнаруженные недостатки в приложении (п.4), составьте план по митигации обнаруженных уязвимостей и недостатков в мобильном приложении
К ДЗ приложите выгруженный в PDF с отчетом MobSF и ваш план по митигации обнаруженных рисков (п.5)

Критерии оценки:
Приложенный PDF с отчетом MobSF
Приложен план по митигации рисков
В плане по митигации рисков (п.2) содержит обнаруженные недостатки и уязвимости, имеется план по их митигации
ДЗ считается принятым, если выполнены все условия.

ㅤ
ㅤ
ㅤ

## ⬩ Развертывание MobSF

Для выполнения работы был использован инструмент **Mobile Security Framework — MobSF**. MobSF предназначен для анализа безопасности мобильных приложений Android, iOS и Windows Mobile и поддерживает как статический, так и динамический анализ. Для данной работы использовался **статический анализ**, так как по условию требовалось провести SAST-тестирование мобильного приложения.

Развертывание MobSF выполнено через Docker.  
Запуск MobSF:

```bash
docker pull opensecurity/mobile-security-framework-mobsf:latest
```

```bash
docker run -it --rm -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
```

После запуска контейнера веб-интерфейс MobSF открыт на `http://127.0.0.1:8000`.

## ⬩ Выбор APK-файла и проведение анализа

**Так как файл из материалов вебинара отсутствовал**, для анализа я взял учебное Android приложение **InsecureBankv2**. Это приложение используется как уязвимый пример для изучения мобильной безопасности.

Для анализа использовался файл `InsecureBankv201.apk`.

После загрузки APK-файла в MobSF инструмент автоматически выполнил статический анализ приложения. В отчете MobSF указаны основные сведения о загруженном файле.

## ⬩ Общие результаты SAST-анализа

По итогам статического анализа MobSF присвоил приложению следующий показатель безопасности:

```text
App Security Score: 27/100
Grade: F
Risk level: CRITICAL RISK
```

Это означает, что приложение имеет **низкий уровень безопасности** и содержит значимые проблемы, требующие исправления. В отчете MobSF также есть распределение находок по уровням серьезности:

```text
High: 8
Medium: 10
Info: 0
Secure: 0
Hotspot: 1
```

В отчете также указана структура приложения:

```text
Activities: 10
Services: 0
Receivers: 2
Providers: 1
Exported Activities: 4
Exported Services: 0
Exported Receivers: 1
Exported Providers: 1
```

---

ㅤ
ㅤ

## 🔸 App Permissions (*со статусом dangerous*)


| PERMISSION                                  | STATUS    | INFO                                             | DESCRIPTION                                                                                                                                                                                                                                      | CODE MAPPINGS |
| ------------------------------------------- | --------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------- |
| `android.permission.ACCESS_COARSE_LOCATION` | dangerous | coarse (network-based) location                  | Access coarse location sources, such as the mobile network database, to determine an approximate phone location, where available. Malicious applications can use this to determine approximately where you are.                                  | —             |
| `android.permission.GET_ACCOUNTS`           | dangerous | list accounts                                    | Allows access to the list of accounts in the Accounts Service.                                                                                                                                                                                   | —             |
| `android.permission.READ_CALL_LOG`          | dangerous | grants read access to the user's call log.       | Allows an application to read the user's call log.                                                                                                                                                                                               | —             |
| `android.permission.READ_CONTACTS`          | dangerous | read contact data                                | Allows an application to read all of the contact (address) data stored on your phone. Malicious applications can use this to send your data to other people.                                                                                     | —             |
| `android.permission.READ_EXTERNAL_STORAGE`  | dangerous | read external storage contents                   | Allows an application to read from external storage.                                                                                                                                                                                             | —             |
| `android.permission.READ_PHONE_STATE`       | dangerous | read phone state and identity                    | Allows the application to access the phone features of the device. An application with this permission can determine the phone number and serial number of this phone, whether a call is active, the number that call is connected to and so on. | —             |
| `android.permission.READ_PROFILE`           | dangerous | read the user's personal profile data            | Allows an application to read the user's personal profile data.                                                                                                                                                                                  | —             |
| `android.permission.SEND_SMS`               | dangerous | send SMS messages                                | Allows application to send SMS messages. Malicious applications may cost you money by sending messages without your confirmation.                                                                                                                | —             |
| `android.permission.USE_CREDENTIALS`        | dangerous | use the authentication credentials of an account | Allows an application to request authentication tokens.                                                                                                                                                                                          | —             |
| `android.permission.WRITE_EXTERNAL_STORAGE` | dangerous | read/modify/delete external storage contents     | Allows an application to write to external storage.                                                                                                                                                                                              |               |


ㅤ

### 🔸 Network Security

В разделе Network Security проблемы уровня High и выше не обнаружены. MobSF показал: `No data available in table`.

ㅤ

## 🔸 Manifest Analysis (*с severity уровня high и выше*)


| NO  | ISSUE                                                                                   | SEVERITY | DESCRIPTION                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | OPTIONS                                                                            |
| --- | --------------------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| 1   | App can be installed on a vulnerable unpatched Android version 4.0.3-4.0.4, [minSdk=15] | high     | This application can be installed on an older version of android that has multiple unfixed vulnerabilities. These devices won't receive reasonable security updates from Google. Support an Android version => 10, API 29 to receive reasonable security updates.                                                                                                                                                                                                                                                                                                        | Suppression the rule vulnerable_os_version in com.android.insecurebankv2.fm.tst    |
| 2   | Debug Enabled For App [android:debuggable=true]                                         | high     | Debugging was enabled on the app which makes it easier for reverse engineers to hook a debugger to it. This allows dumping a stack trace and accessing debugging helper classes.                                                                                                                                                                                                                                                                                                                                                                                         | Suppression the rule app_is_debuggable in com.android.insecurebankv2.fm.tst        |
| 4   | Activity (com.android.insecurebankv2.PostLogin) is vulnerable to StrandHogg 2.0         | high     | Activity is found to be vulnerable to StrandHogg 2.0 task hijacking vulnerability. When vulnerable, it is possible for other applications to place a malicious activity on top of the activity stack of the vulnerable application. This makes the application an easy target for phishing attacks. The vulnerability can be remediated by setting the launch mode attribute to "singleInstance" and by setting an empty taskAffinity (taskAffinity=""). You can also update the target SDK version (26) of the app to 29 or higher to fix this issue at platform level. | Suppression the rule activity_task_hijacking2 in com.android.insecurebankv2.fm.tst |
| 6   | Activity (com.android.insecurebankv2.DoTransfer) is vulnerable to StrandHogg 2.0        | high     | Activity is found to be vulnerable to StrandHogg 2.0 task hijacking vulnerability. When vulnerable, it is possible for other applications to place a malicious activity on top of the activity stack of the vulnerable application. This makes the application an easy target for phishing attacks. The vulnerability can be remediated by setting the launch mode attribute to "singleInstance" and by setting an empty taskAffinity (taskAffinity=""). You can also update the target SDK version (26) of the app to 29 or higher to fix this issue at platform level. | Suppression the rule activity_task_hijacking2 in com.android.insecurebankv2.fm.tst |
| 8   | Activity (com.android.insecurebankv2.ViewStatement) is vulnerable to StrandHogg 2.0     | high     | Activity is found to be vulnerable to StrandHogg 2.0 task hijacking vulnerability. When vulnerable, it is possible for other applications to place a malicious activity on top of the activity stack of the vulnerable application. This makes the application an easy target for phishing attacks. The vulnerability can be remediated by setting the launch mode attribute to "singleInstance" and by setting an empty taskAffinity (taskAffinity=""). You can also update the target SDK version (26) of the app to 29 or higher to fix this issue at platform level. | Suppression the rule activity_task_hijacking2 in com.android.insecurebankv2.fm.tst |
| 12  | Activity (com.android.insecurebankv2.ChangePassword) is vulnerable to StrandHogg 2.0    | high     | Activity is found to be vulnerable to StrandHogg 2.0 task hijacking vulnerability. When vulnerable, it is possible for other applications to place a malicious activity on top of the activity stack of the vulnerable application. This makes the application an easy target for phishing attacks. The vulnerability can be remediated by setting the launch mode attribute to "singleInstance" and by setting an empty taskAffinity (taskAffinity=""). You can also update the target SDK version (26) of the app to 29 or higher to fix this issue at platform level. | Suppression the rule activity_task_hijacking2 in com.android.insecurebankv2.fm.tst |


ㅤ

### 🔸 Code Analysis (*с severity уровня high и выше*)

В разделе Code Analysis проблемы уровня High и выше не обнаружены. MobSF показал: `No data available in table`.

---

ㅤ
ㅤ
ㅤ
ㅤ
ㅤ
ㅤ
ㅤ
ㅤ
ㅤ
ㅤ

## 📈 План по митигации обнаруженных рисков

По результатам статического анализа APK-файла `InsecureBankv201.apk` в MobSF приложение получило Security Score `**27/100`**, что соответствует **критическому уровню** риска. В отчете также указано 8 находок уровня High, 10 находок уровня Medium и 1 Hotspot.

ㅤ

## 🔹 1. Dangerous Permissions

В приложении обнаружены опасные разрешения Android: `WRITE_EXTERNAL_STORAGE`, `SEND_SMS`, `USE_CREDENTIALS`, `GET_ACCOUNTS`, `READ_PROFILE`, `READ_CONTACTS`, `READ_PHONE_STATE`, `READ_EXTERNAL_STORAGE`, `READ_CALL_LOG`, `ACCESS_COARSE_LOCATION`. Эти разрешения дают доступ к чувствительным данным пользователя или позволяют выполнять потенциально опасные действия — например читать контакты, журнал звонков, состояние телефона, местоположение, внешнее хранилище или отправлять SMS.

Android относит `dangerous / runtime permissions` к разрешениям, которые дают доступ к ограниченным данным или позволяют выполнять чувствительные действия, поэтому приложение должно запрашивать их явно и только перед конкретным действием, где они действительно нужны.

**⬩ План митигации ⬩ :**

1. Провести ревизию всех dangerous permissions и удалить из `AndroidManifest.xml` разрешения, которые не нужны для основной логики приложения.
2. Не запрашивать **все** разрешения при первом запуске приложения. Запрашивать каждое разрешение только в момент, когда пользователь запускает функцию, которой это разрешение действительно необходимо.
3. Для `SEND_SMS` — отказаться от прямой отправки SMS из приложения, если это не является обязательной функцией. Если функция нужна, использовать системный SMS intent, чтобы пользователь сам подтверждал отправку.
4. Для `READ_CONTACTS`, `READ_CALL_LOG`, `READ_PHONE_STATE`, `GET_ACCOUNTS`, `READ_PROFILE` — ограничить доступ к персональным данным и заменить такие механизм⬩ы менее чувствительными альтернативами.
5. Для `READ_EXTERNAL_STORAGE` и `WRITE_EXTERNAL_STORAGE` — перейти на более ограниченные механизмы доступа к файлам, например `scoped storage` или системный выбор файла.
6. Для `ACCESS_COARSE_LOCATION` — запрашивать доступ к местоположению только при наличии функции, которая реально зависит от **геолокации**.
7. Реализовать корректную обработку отказа пользователя от выдачи разрешения: приложение не должно падать, а должно продолжать работать с ограниченным функционалом.
8. Добавить проверку разрешений перед каждым обращением к чувствительным данным.

ㅤ

## 🔹 2. Network Security

В разделе `Network Security` MobSF **не обнаружил проблем уровня High** и выше: таблица в отчете пустая.

**⬩ План митигации ⬩ :**

1. Несмотря на отсутствие находок, оставить обязательное требование использовать HTTPS для всех сетевых соединений.
2. Запретить `cleartext traffic` через `Network Security Config`, если приложению не требуется HTTP для тестового стенда.
3. Проверить, что тестовые домены, IP-адреса и временные `backend endpointы` не остаются в релизной версии приложения.
4. Использовать актуальные TLS-настройки и не отключать проверку сертификатов.
5. Добавить проверку сетевой конфигурации в процесс сборки приложения, чтобы новые HTTP endpointы не попадали в релиз.

ㅤ

## 🔹 3. Manifest Analysis *— устаревший minSdk*

MobSF обнаружил, что приложение может быть установлено на старую уязвимую версию Android `4.0.3–4.0.4`, потому что указан `minSdk=15`. В отчете указано, что такие устройства имеют неисправленные уязвимости и не получают разумные security updates.  
Google Play требует, чтобы новые приложения и обновления таргетировали актуальные уровни API; это нужно, чтобы пользователи получали преимущества новых механизмов безопасности, приватности и производительности.

**⬩ План митигации ⬩ :**

1. Повысить минимально поддерживаемую версию Android до более актуального уровня.
2. Обновить `targetSdkVersion` до актуального значения, соответствующего требованиям Google Play.
3. Провести тестирование приложения на новых версиях Android после повышения SDK.
4. Удалить или переписать устаревшую логику, которая была нужна только для поддержки старых Android-версий.
5. Ввести политику регулярного обновления `minSdkVersion` и `targetSdkVersion`.

ㅤ

## 🔹 4. Manifest Analysis *— включен debug-режим*

MobSF обнаружил `android:debuggable=true`, то есть приложение собрано с включенной отладкой. В отчете указано, что это облегчает reverse engineering, подключение debugger, получение stack trace и доступ к debug helper classes.  
`android:debuggable=true` позволяет атакующему отлаживать приложение и упрощает доступ к частям приложения, которые должны оставаться защищенными.

**⬩ План митигации ⬩ :**

1. Отключить `android:debuggable` в релизной сборке.
2. Разделить `debug`- и `release`- конфигурации сборки.
3. Проверить, что release-сборка подписывается отдельно и не содержит debug-флагов.
4. Добавить автоматическую проверку в `CI/CD`: если в release-сборке обнаружен `android:debuggable=true`, сборка должна завершаться ошибкой.
5. Удалить отладочные логи, тестовые классы и debug helper-код из релизной версии приложения.

ㅤ

## 🔹 5. Manifest Analysis *— StrandHogg 2.0*

MobSF обнаружил уязвимость **StrandHogg 2.0** у нескольких Activity: `PostLogin`, `DoTransfer`, `ViewStatement`, `ChangePassword`. В отчете указано, что такая проблема может позволить другому приложению поместить вредоносную `Activity` поверх `Activity` уязвимого приложения, что создает риск `phishing`-атаки и `task hijacking`.

**⬩ План митигации ⬩ :**

1. Для уязвимых Activity настроить `launchMode="singleInstance"`.
2. Установить пустой `taskAffinity=""` для Activity, которые не должны использовать общую task affinity.
3. Обновить `targetSdkVersion` до 29 или выше, так как в отчете MobSF указано, что это может закрыть проблему на уровне платформы.
4. Перепроверить все Activity, связанные с аутентификацией, переводами, просмотром счетов и сменой пароля.
5. Запретить запуск чувствительных Activity извне, если для этого нет бизнес-необходимости.
6. После исправления повторно просканировать APK в MobSF и убедиться, что StrandHogg 2.0 больше не отображается в High-находках.

ㅤ

## 🔹 6. Exported components

В отчете указано несколько компонентов с `android:exported=true`: Activity `PostLogin`, `DoTransfer`, `ViewStatement`, `ChangePassword`, Content Provider `TrackUserContentProvider`, Broadcast Receiver `MyBroadCastReceiver`. Эти пункты в отчете имеют уровень **warning**, но их нужно учитывать в плане митигации, потому что exported-компоненты доступны другим приложениям на устройстве.

`android:exported` определяет, может ли Activity, Service или Receiver запускаться компонентами других приложений; при `true` компонент может быть доступен внешним приложениям.

**⬩ План митигации ⬩ :**

1. Для всех внутренних Activity установить `android:exported="false"`.
2. Оставить `android:exported="true"` только для компонентов, которые действительно должны быть доступны извне.
3. Для exported-компонентов добавить permission-защиту.
4. Для Content Provider ограничить доступ через permissions и запретить внешнее чтение, если оно не нужно.
5. Для Broadcast Receiver проверить, какие intent принимает компонент, и ограничить прием внешних broadcast-сообщений.
6. Проверить, что чувствительные экраны — вход, перевод денег, просмотр выписки, смена пароля — не могут быть вызваны напрямую из другого приложения.

ㅤ

## 🔹 7. Certificate Analysis *— debug certificate*

MobSF обнаружил, что приложение подписано **debug certificate**. В отчете указано, что прод приложение не должно распространяться с **debug certificate**.

**⬩ План митигации ⬩ :**

1. Переподписать релизную сборку production-ключом.
2. Хранить production `signing key` отдельно от исходного кода.
3. Не использовать `debug` keystore для релизных APK.
4. Разделить `debug` и `release` signing configs.
5. Добавить автоматическую проверку, которая блокирует релиз, если APK подписан debug-сертификатом.

ㅤ

## 🔹 8. Certificate Analysis *— SHA1withRSA*

MobSF обнаружил, что приложение подписано с использованием `SHA1withRSA`; в отчете указано, что SHA1 известен проблемами с коллизиями.

**⬩ План митигации ⬩ :**

1. Использовать более стойкий алгоритм подписи, например **SHA-256**.
2. Перевыпустить приложение с новым production-сертификатом.
3. Проверить, что релизная подпись поддерживает современные схемы подписи APK.
4. Не использовать устаревшие алгоритмы хеширования для подписи новых сборок.
5. Повторно проверить сертификат через `MobSF` после переподписания.

ㅤ

## 🔹 9. Code Analysis

В разделе `Code Analysis` **MobSF не обнаружил проблем уровня High** и выше. В отчете таблица Code Analysis пустая.

При этом MobSF отдельно показал блок `Hardcoded Secrets` с возможными секретами и строками высокой энтропии. Эти значения требуют ручной проверки, потому что часть из них может быть ложными срабатываниями, но хранение реальных ключей, токенов и паролей внутри APK является опасной практикой.

**⬩ План митигации ⬩ :**

1. Проверить все строки из блока `Hardcoded Secrets`.
2. Удалить реальные пароли, токены, ключи API и криптографические ключи из кода приложения.
3. Не хранить секреты внутри APK, потому что APK можно декомпилировать и извлечь строки.
4. Для чувствительных ключей использовать серверную сторону или защищенное хранилище.
5. Все найденные реальные секреты считать скомпрометированными и заменить.
6. Добавить проверку секретов в `CI/CD`, чтобы новые ключи не попадали в исходный код и APK.

---

ㅤ

### Общий вывод

По результатам анализа MobSF основные риски приложения связаны не с Network Security и не с Code Analysis, где High-находок не было — а с конфигурацией Android-приложения: опасные разрешения, устаревшая минимальная версия Android, включенный debug-режим, уязвимость Activity к StrandHogg 2.0, debug-сертификат и устаревший алгоритм подписи.  
Основной план митигации заключается в ужесточении Android Manifest, отключении debug-настроек, обновлении SDK, ограничении exported-компонентов, пересборке release-версии с production-сертификатом и повторном сканировании APK в MobSF после исправлений.