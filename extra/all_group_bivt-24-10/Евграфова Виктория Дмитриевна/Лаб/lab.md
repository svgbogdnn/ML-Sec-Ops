Часть 1

whoami до запуска контейнера
![alt text](image-1.png)

запуск командной оболочки sh внутри контейнера docker run -it ket9/otus-devsecops sh
и команда whoami внутри контейнера
![alt text](image.png)

Часть 2

![alt text](image-2.png)

Dockerfile:
![alt text](image-3.png)

Часть 3

Часть вывода сканера:
![alt text](image-5.png)
![alt text](image-6.png)

Вывод: 
Образ f содержит критические и высокие уязвимости в системных библиотеках. 
Некоторые выявленные проблемы:
CVE-2016-9841 (CRITICAL) 
CVE-2017-18078 (HIGH)
CVE-2016-2779 (HIGH)
Злоумышленник сможет повышать привилегии до root и вызывать отказ в обслуживании