# Лабораторная работа №6

## Выполнение

Запустил контейнер с Python:  
docker run -it -p 8080:8080 --name trufflehog-lab python:buster /bin/bash  
Внутри установил truffleHog:  
pip install truffleHog  
Просканировал репозиторий и сохранил лог:  
trufflehog https://github.com/OtusTeam/DevSecOps_secret-finding.git | tee -a log.txt  

## Найденные секреты

1. Токен Codacy API в файле README.MD   
   Строка: b69ee3a86e3b4afcaf993f210fccfb1d  
   Это API-токен сервиса Codacy, используется для бейджа статуса проекта. Если попадёт не в те руки, даст доступ к аналитике и данным о покрытии кода.  
   Правильное хранение: хранить в переменных окружения CI/CD, в коде заменять на плейсхолдер.

2. JWT-токен в файле webgoat-lessons/jwt/src/main/resources/lessonPlans/en/JWT_weak_keys.adoc  
   Строка: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJXZWJHb2F0IFRva2VuIEJ1aWxkZXIiLCJpYXQiOjE1MjQyMTA5MDQsImV4cCI6MTYxODkwNTMwNCwiYXVkIjoid2ViZ29hdC5vcmciLCJzdWIiOiJ0b21Ad2ViZ29hdC5jb20iLCJ1c2VybmFtZSI6IlRvbSIsIkVtYWlsIjoidG9tQHdlYmdvYXQuY29tIiwiUm9sZSI6WyJNYW5hZ2VyIiwiUHJvamVjdCBBZG1pbmlzdHJhdG9yIl19.vPe-mqQPOt78zK8wrbN1TjNJj3LeX9Qbch6oo23RUJgM  
   Готовый подписанный JWT, который использовался для тестов. Внутри есть имя пользователя, роли и срок действия. Если кто-то получит этот токен, сможет выдать себя за пользователя Tom с правами администратора.  
   Правильное хранение: генерировать токены на лету в тестах, не хранить подписанные JWT в репозитории.

3. Зашифрованный GitHub-токен в файле .travis.yml  
   Строка: secure: pJOLBnl6427PcVg/tVy/qB18JC7b8cKpffau+IP0pjdSt7KUfBdBY3QuJ7mrM65zRoVILzggLckaew2PlRmYQRdumyWlyRn44XiJ9KO4n6Bsufbz+ictB4ggtozpp9+I9IIUh1TmqypL9lhkX2ONM9dSHmyblYpAAgMuYSK8FYc=  
   Зашифрованная переменная Travis CI с токеном для создания GitHub-релизов. Если кто-то доберётся до логов сборки, он может попробовать расшифровать этот токен и создавать релизы от имени проекта.  
   Правильное хранение: не хранить секреты в репозитории, даже если они зашифрованы средствами CI. Использовать временные токены и настройки CI/CD.

4. Зашифрованный Heroku API-ключ в файле .travis.yml  
   Строка: secure: eqSm5syJhyvIwxQ/ZCMtfFVayiZjsr+1m0eIR36FKMU6iSz5V351G+VNjCy/G+7EIsm+KuoLHqbl+NxmmOsDf2YoQk8KAdnbecMLWVwB+VncLM0ZU4mEEBt3lJWUzStoy9UNgzKs6Nc/HQ0zllV61NfgFS17pNHvce5WfjKHzTA=  
   Зашифрованный ключ для деплоя на Heroku. Даёт полный контроль над приложением, включая возможность загружать новые версии и менять настройки.  
   Правильное хранение: использовать Heroku API key с ограниченным сроком, задавать через переменные окружения в настройках CI.

5. Зашифрованный Slack-вебхук в файле .travis.yml  
   Строка: secure: S9VFew5NSE8WDzYD1VDBUULKKT0fzgblQACznwQ85699b2yeX9TX58N3RZvRS1JVagVP1wu2xOrwN2g+AWx4Ro3UBZD5XG86uTJWpCLD4cRWHBoGMH2TfvI7/IzsWmgxH4MBxFRvZr/eEhlVAux+N9H4EoEdS4CKsJXEqV37PlA=  
   Зашифрованный URL для отправки уведомлений в Slack. Если его расшифровать, можно отправлять сообщения в рабочий канал от имени CI, в том числе спам или фишинговые ссылки.  
   Правильное хранение: хранить вебхук только в настройках CI/CD, не добавлять в файлы конфигурации, даже в зашифрованном виде.

## Выводы

trufflehog находит в истории Git разные типы секретов: токены API, JWT, зашифрованные переменные CI/CD для GitHub, Heroku и Slack. Даже если секрет удалили из текущей версии кода, он остаётся в истории коммитов и может быть найден. Чтобы такого не случалось, нельзя коммитить секреты в репозиторий. Нужно использовать переменные окружения, внешние хранилища (Hashicorp Vault) и регулярно сканировать код. Инструмент можно встроить в CI/CD пайплайн для автоматической проверки при каждом push.