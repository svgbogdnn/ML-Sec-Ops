## Отчет по части III

Для сканирования образа использовался Trivy:

```bash
docker run --rm aquasec/trivy:latest image --severity HIGH,CRITICAL ket9/otus-devsecops-owasp-rest:latest
```

Сканировался образ:

```text
ket9/otus-devsecops-owasp-rest:latest
```

Результаты сканирования сохранены на скриншотах:

- `report_summary.png` - сводка Trivy;
- `report_2.png` - уязвимости системной части Alpine;
- `report_3.png` - уязвимости Python-зависимостей.

По сводке Trivy для `alpine 3.14.2` найдено 56 уязвимостей:

```text
Total: 56 (HIGH: 48, CRITICAL: 8)
```

Для Python-зависимостей найдено 14 уязвимостей высокого уровня:

```text
Total: 14 (HIGH: 14, CRITICAL: 0)
```

Проблемы есть не только в приложении, но и в самом окружении:

- устаревшая база `alpine 3.14.2`;
- системные пакеты, например `busybox`, `expat`, `xz-libs`, `zlib`;
- Python-зависимости, например `Flask`, `PyJWT`, `Werkzeug`, `certifi`, `setuptools`, `urllib3`, `wheel`.

Вывод: контейнер не стоит использовать в текущем виде. Нужно обновить базовый образ, системные пакеты и Python-зависимости, затем пересобрать образ и повторно проверить его через Trivy.
