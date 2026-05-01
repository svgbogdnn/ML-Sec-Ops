# Чаcть 1

Запустим обычный alpine. В системе пользователь - мой, внутри контейнера - root.

# Часть 2

Возьмем тот же alpine, создадим юзера alpineuser и добавим его в группу alpinegroup. USER alpineuser делает так, что внутри контейнера мы больше не  являемся рутом.

Из флагов для безопасности отключим все capabilities, интернет и отключим повышение привилегий.

Зайдем в контейнер и проверим, что мы не рут и что сеть отключена.

# Часть 3

**Образ имеет 0 уязвимостей, что неудивительно, так как версия alpine максимально свежая: 3.23.4. (trivy image alpine:latest дает такой же вывод)**

~/Downloads$ trivy image alpinesecure:latest                  
2026-04-25T19:50:26+03:00  INFO  [vuln] Vulnerability scanning is enabled
2026-04-25T19:50:26+03:00  INFO  [secret] Secret scanning is enabled
2026-04-25T19:50:26+03:00  INFO  [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2026-04-25T19:50:26+03:00  INFO  [secret] Please see https://trivy.dev/docs/v0.70/guide/scanner/secret#recommendation for faster secret detection
2026-04-25T19:50:26+03:00  INFO  Detected OS  family="alpine" version="3.23.4"
2026-04-25T19:50:26+03:00  INFO  [alpine] Detecting vulnerabilities...  os_version="3.23" repository="3.23" pkg_num=16
2026-04-25T19:50:26+03:00  INFO  Number of language-specific files  num=0

Report Summary

┌─────────────────────────────────────┬────────┬─────────────────┬─────────┐
│               Target                │  Type  │ Vulnerabilities │ Secrets │
├─────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ alpinesecure:latest (alpine 3.23.4) │ alpine │        0        │    -    │
└─────────────────────────────────────┴────────┴─────────────────┴─────────┘
Legend:

- '-': Not scanned
- '0': Clean (no security findings detected)

**Для примера ket9/opus-devsecops-owasp-rest:latest имеет 72 уязвимости в alpine из-за несвежей версии 3.14.2**
~/Downloads$ trivy image ket9/otus-devsecops-owasp-rest:latest
2026-04-25T20:01:12+03:00  INFO  [vuln] Vulnerability scanning is enabled
2026-04-25T20:01:12+03:00  INFO  [secret] Secret scanning is enabled
2026-04-25T20:01:12+03:00  INFO  [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2026-04-25T20:01:12+03:00  INFO  [secret] Please see https://trivy.dev/docs/v0.70/guide/scanner/secret#recommendation for faster secret detection
2026-04-25T20:01:12+03:00  INFO  Detected OS  family="alpine" version="3.14.2"
2026-04-25T20:01:12+03:00  INFO  [alpine] Detecting vulnerabilities...  os_version="3.14" repository="3.14" pkg_num=57
2026-04-25T20:01:12+03:00  INFO  Number of language-specific files  num=1
2026-04-25T20:01:12+03:00  INFO  [python-pkg] Detecting vulnerabilities...
2026-04-25T20:01:12+03:00  WARN  This OS version is no longer supported by the distribution  family="alpine" version="3.14.2"
2026-04-25T20:01:12+03:00  WARN  The vulnerability detection may be insufficient because security updates are not provided
2026-04-25T20:01:12+03:00  INFO  Table result includes only package filenames. Use '--format json' option to get the full path to the package file.

Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬────────────┬─────────────────┬─────────┐
│                                      Target                                      │    Type    │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼────────────┼─────────────────┼─────────┤
│ ket9/otus-devsecops-owasp-rest:latest (alpine 3.14.2)                            │   alpine   │       72        │    -    │ 





**Вывод: образ пока не имеет известных CVE, но это ничего не значит: во-первых в нем не запущено никакого приложения, во-вторых всё равно нужно отключить лишние функции (capabilities, root), чтобы если уязвимости и были найдены в последней версии alpine или запущенном приложении, то их было сложнее эксплуатировать.**