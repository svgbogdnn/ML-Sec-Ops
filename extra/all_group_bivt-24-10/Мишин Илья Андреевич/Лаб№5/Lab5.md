# Часть 1
Образы на компьютере
![[Pasted image 20260508074654.png]]
Пользователь на компьютере 
![[CleanShot 2026-05-08 at 07.47.46@2x.png]]
Пользователь в контейнере *nginx:1.27-alpine*
![[CleanShot 2026-05-08 at 07.54.17@2x.png]]

# Часть 2
Составленный образ
```
FROM python:3.11-slim  
  
RUN groupadd -r testgr && \  
    useradd -r -g testgr test  
  
USER test  
  
CMD ["python3", "--version"]

# Дополнительные настройки безопасности:  
# 1. Запускать с read-only файловой системой:  
#    docker run --read-only  
# 2. Ограничить cpu и memory:  
#    docker run --memory=512m --cpus=1  
# 3. Запретить повышение привилегий:  
#    docker run --security-opt=no-new-privileges:true  
# 4. Ограничить capabilities:  
#    docker run --cap-drop=ALL
```
Результаты выполенения
![[CleanShot 2026-05-08 at 08.10.15@2x.png]]

# Часть 3
Проверять образ *my-python-nonroot* буду через trivy
Исполнив команду 
```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \      
  aquasec/trivy image my-python-nonroot
```
Получил следущее
![[CleanShot 2026-05-08 at 08.20.09@2x.png]]
![[CleanShot 2026-05-08 at 08.20.46@2x.png]]
![[CleanShot 2026-05-08 at 08.21.13@2x.png]]
и так еще несколько проблем
![[CleanShot 2026-05-08 at 08.21.49@2x.png]]

Проанализировав вывод сканера trivy - можно сделать следующие выводы:
После сканирования Trivy обнаружил 238 уязвимостей:
- CRITICAL: 0
- HIGH: 20 (8.4%)
- MEDIUM: 90 (37.8%)
- LOW: 128 (53.8%)
Уязвимости распределены между Debian 13.4 (112 шт.) и Python-пакетами (126 шт.).
Можно заметить, что основные проблемы возникли в устаревших пакетах. Также из положительного- нет критичных уязвимостей  
