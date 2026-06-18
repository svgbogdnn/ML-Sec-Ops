# Лабораторная работа №6
## Секрет 1
```
env:
    GITHUB_TOKEN: cee3257f0e59d1d4975a35429cffc3d8b5fabe15ca2e9d47243cbc446d4894a5
```
### Расположение
.github/workflows/release.yml : 74
### Объяснение
Секретный токен для github. Его раскрытие может дать злоумышленникам доступ к репо / аккаунту (маловероятно, но всё же).
### Как хранить
Либо в разделе для секретов CI/CD окружения, либо (вариант попроще) в переменных окружения.

## Секрет 2
```
------BEGIN PRIVATE KEY-----
-MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCpuOH6AnMc/xdJ
-Fo/8z5j+MTLS1yVHANOgQgAxrIczt9uS9vi9UtE0377fL0TQs5Uo97rSuS6ZJ0ac
-WKhNEMBFhR3unINs1FC9PzEjI7UDnSacZNg19eCsAn8hdO88TTt5YhIsPZyg0E29
-KbzuLiBhSaKFmnmGbh+T4l9ZIy/t1BcOfzFOpvhEMIriRgA3asZpg8CnNosvbO2A
-lZ6Z7nFOibJ1hrzQQgl2Ev+nf5VRxo34UGGbF+65mCGFmBvOYn+IbsciADT+2/4T
-gr3aVWD5zwn2NRQI5FKpFePVArpfI85Lxt4eHYzpTWI5dYd++H1BesjoT0ZEM0DP
-IVfvMN4XAgMBAAECggEAfkD6WXDZEQjx2XzfP8FunikbFZzWLit/QgfW/RzKtr5e
-qMTv5GZnGl4XLw+KsXXlz8P5RihbcbK15DhPeoSrgwuzaH0lhx+psB4B/5HgZf/R
-aSXbcMiniU2SJOFH1iPdyj4aJq7uhPJv4ffag5PsonKUY662GDpzYx9SroxuawvU
-kUyHGnXP+QN3obFbHUqVx72GhwbyvKUXU4wMdQob+QRmB1excNCGVoGDfKVVOev6
-S38+rDCbHo0gK+4UF4ljlJAhbZwQTvv7LoXJu27kUlAvGObjFY8hfbs9T2BStnmh
-8D87TMp77gAhymqB+OS6OQrbnO/4HG7L+hgougmfgQKBgQDXYycFMwRoR4qyeKNa
-mGxQghvsfJdaWO2axS5RikPi5YkpIfYwuUKLL/niRwCslBz20iFihIJS2dWuOLYD
-OqAXUbUDAMbjYioWoMNDp6D3Y3eqSkmMAU3Ra+qBusQmp0Ffcr0GNL5xnsu1nmO+
-jOOBZ7YD5Hry5oIjvfDmBMTumwKBgQDJuXQbGYz/0d12Feg616zyqDUXcLSQWK7F
-nz1140VPdwfvWHMYWomUats5yMxycuGGDe/DH779rKJHT9U9xGsjlSQvLex0QMy8
-ddn7vvfLoPzhQx6NAVvobaobY2+MSgJmOedrd2n7p/RGTTSAwdIsghy2BNxdYgZD
-P4uBMf3oNQKBgQDIS7OQuTXDB6yqdVdru00WvTfcfqx9Xy9ueymsuEiTKuOXdbas
-7ss8Bpx5WY/97SrWOOjE5fcPtvVoM+LHM/CGXvxW05UhBTugmVWch7k/9ablnHmy
-kc/dDV8hzx3z2BwJ9/hiWhA0Nvi3Z5jYLcqvn1N7YTEYy1WAiXUJYqwEOwKBgQCM
-m3D7pr6qXi1Arxp1UXoile6TzSJ+7uG7rDhZ4LWiIzTrtzpaglkdk7IFQBqJt9vM
-5g/2cT1ecqOWk2XurOeFIOLc4+TKT5Sl1HvBxyXP0QITPgaggI8AntgQSSoqnje3
-66qMNOsx16skCZKMIQ2Pqo26rf6wNLBq1XM29ZKm9QKBgGeefgyYgirI1TdjpqVk
-B5GhVf+D4wozBvJFAwmVBAQFFU1wCJV7aiwT/RL9KP21Ahfil5Ll7OHE80S4yRU2
-g5Y5/F3ExwrIUWdnnMgsO3VtmpgBR5ADBbMQ2Wyo8VF9+tENtlFfnRDRFZl09x1H
-oX44T7mclitCYaOuoRnC2V5H
------END PRIVATE KEY-----
```
### Расположение
webgoat-server/privatekey.key : 1
### Объяснение
Насколько я понимаю, это целый файл с RSA ключом. Доступ к нему даёт возможность расшифровывать трафик к серверу.
### Как хранить
Можно шифровать, можно передавать между сотрудниками в отдельном контуре, можно пользоваться специальными системами для хранения и работы с подобными секретами.


## Секрет 3
```
-public interface SolutionConstants {
-
-    //TODO should be random generated when starting the server
-    String PASSWORD = "!!webgoat_admin_1234!!";
-    String PASSWORD_TOM = "thisisasecretfortomonly";
-    String ADMIN_PASSWORD_LINK = "375afe1104f4a487a73823c50a9292a2";
-}
```
### Расположение
webgoat-lessons/challenge/src/main/java/org/owasp/webgoat/plugin/SolutionConstants.java : 37
### Объяснение
Насколько я понимаю, это credentials для дженерик аккаунта с админским доступом. Соотвественно, злоумышленник, получивший доступ к коду, может легко использовать этот аккаунт.
### Как хранить
Как там, собственно, и написано, здесь нужно генерировать случайные данные каждый раз при старте сервера. А в целом подобные секреты обычно хранятся переменных окружения.


## Секрет 4
```
 [![Codacy Badge](https://api.codacy.com/project/badge/b69ee3a86e3b4afcaf993f210fccfb1d)](https://www.codacy.com/app/dm/WebGoat)
```
### Расположение
README.MD : 3
### Объяснение
Это ключ (вполне возможно, закрытый) к стороннему API. Его утечка может означать слив данных, или, если это какой-то платный сервис, можно знатно попасть на деньги:\)
### Как хранить
Либо в разделе для секретов CI/CD окружения, либо (вариант попроще) в переменных окружения.


## Секрет 5
```
api_key:
  secure: pJOLBnl6427PcVg/tVy/qB18JC7b8cKpffau+IP0pjdSt7KUfBdBY3QuJ7mrM65zRoVILzggLckaew2PlRmYQRdumyWlyRn44XiJ9KO4n6Bsufbz+ictB4ggtozpp9+I9IIUh1TmqypL9lhkX2ONM9dSHmyblYpAAgMuYSK8FYc=
file_glob: true
```
### Расположение
Filepath.travis.yml : 23
### Объяснение
Судя по всему, это клоч для доступа к пайплайну CI/CD. Злоумышленник может получить возможность добавления вредоносного кода в программу, и/или получения информации об репозитории для выполнения дальнейших атак.
### Как хранить
Либо в разделе для секретов CI/CD окружения, либо (вариант попроще) в переменных окружения.