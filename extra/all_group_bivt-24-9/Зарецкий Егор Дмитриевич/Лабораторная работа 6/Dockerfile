FROM ket9/otus-devsecops-xss:latest


RUN groupadd -g 10001 appgroup \
    && useradd -m -u 10001 -g appgroup -s /usr/sbin/nologin appuser \
    && chown -R appuser:appgroup /var/www/html

RUN mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 \
    && chown -R appuser:appgroup /var/lock/apache2 /var/run/apache2 /var/log/apache2

EXPOSE 80

# инструция по запуску 
# 1 запрещает записьв корневую файловую систему --read-only
# 2 отключает все линукс привилегии, оставляем необхзодимое:
#    --cap-drop=ALL --cap-add=NET_BIND_SERVICE
# 3 запрет повышение привилегий:
#    --security-opt no-new-privileges:true
# 4 ограничение по памяти и ресурсам:
#    --memory=256m --cpus=1 --pids-limit=200
# 5  изолированная сеть:
#    --network bridge (или dedicated user-defined network)

USER appuser 
#как раз таки запуск не от рута

CMD ["sh", "-c", "whoami && sleep 3600"]
