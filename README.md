
# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»

------

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Решщение 1

![Балансировка1.1](https://github.com/zlodey-paha/10-2/blob/main/10-2/1.1.%20%D0%91%D0%B0%D0%BB%D0%B0%D0%BD%D1%81%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0.PNG)
![Балансировка1.2](https://github.com/zlodey-paha/10-2/blob/main/10-2/1.2.%20%D0%91%D0%B0%D0%BB%D0%B0%D0%BD%D1%81%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0.PNG)
![HAProxy.cfg](https://github.com/zlodey-paha/10-2/blob/main/10-2/1.3.%20haproxy.cfg)
HAProxy.cfg
```
global
    daemon
    maxconn 256
    log /dev/log local0

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms
    option forwardfor
    option http-server-close
    log global

listen stats
    bind *:1936
    mode http
    stats enable
    stats hide-version
    stats refresh 30s
    stats show-node
    stats auth admin:password
    stats uri /haproxy?stats

frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /
    
    server server1 127.0.0.1:8081 check inter 2000 rise 2 fall 3
    server server2 127.0.0.1:8082 check inter 2000 rise 2 fall 3
```

------

### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

### Решщение 2

![Балансировка2.1](https://github.com/zlodey-paha/10-2/blob/main/10-2/2.1.%20%D0%91%D0%B0%D0%BB%D0%B0%D0%BD%D1%81%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0.PNG)
![Балансировка2.2](https://github.com/zlodey-paha/10-2/blob/main/10-2/2.2.%20%D0%91%D0%B0%D0%BB%D0%B0%D0%BD%D1%81%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0.PNG)
![HAProxy.cfg](https://github.com/zlodey-paha/10-2/blob/main/10-2/2.3.%20haproxy.cfg)
HAProxy.cfg
```
global
    daemon
    maxconn 4000
    log /dev/log local0 info
    stats socket /var/run/haproxy/admin.sock mode 660 level admin

defaults
    mode http
    log global
    option httplog
    option dontlognull
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms
    retries 3
    option redispatch

listen stats
    bind *:1936
    mode http
    stats enable
    stats hide-version
    stats refresh 30s
    stats show-node
    stats auth admin:password
    stats uri /haproxy?stats

frontend http_front
    bind *:80
    mode http
    
    acl is_example_local hdr(host) -i example.local
    
    use_backend http_servers if is_example_local
    
    default_backend invalid_domain

backend http_servers
    mode http
    balance roundrobin
    option httpchk GET /health
    
    server server1 127.0.0.1:8081 weight 2 check inter 2000 fall 3 rise 2
    server server2 127.0.0.1:8082 weight 3 check inter 2000 fall 3 rise 2  
    server server3 127.0.0.1:8083 weight 4 check inter 2000 fall 3 rise 2
    
    capture request header Host len 32
    capture request header User-Agent len 64
    http-request capture req.hdr(X-Forwarded-For) len 15

backend invalid_domain
    mode http
    errorfile 503 /etc/haproxy/errors/403.http

backend direct_access
    mode http
    server direct_server 127.0.0.1:8080
```

---

## Задания со звёздочкой*
Эти задания дополнительные. Их можно не выполнять. На зачёт это не повлияет. Вы можете их выполнить, если хотите глубже разобраться в материале.

---

### Задание 3*
- Настройте связку HAProxy + Nginx как было показано на лекции.
- Настройте Nginx так, чтобы файлы .jpg выдавались самим Nginx (предварительно разместите несколько тестовых картинок в директории /var/www/), а остальные запросы переадресовывались на HAProxy, который в свою очередь переадресовывал их на два Simple Python server.
- На проверку направьте конфигурационные файлы nginx, HAProxy, скриншоты с запросами jpg картинок и других файлов на Simple Python Server, демонстрирующие корректную настройку.

---

### Задание 4*
- Запустите 4 simple python сервера на разных портах.
- Первые два сервера будут выдавать страницу index.html вашего сайта example1.local (в файле index.html напишите example1.local)
- Вторые два сервера будут выдавать страницу index.html вашего сайта example2.local (в файле index.html напишите example2.local)
- Настройте два бэкенда HAProxy
- Настройте фронтенд HAProxy так, чтобы в зависимости от запрашиваемого сайта example1.local или example2.local запросы перенаправлялись на разные бэкенды HAProxy
- На проверку направьте конфигурационный файл HAProxy, скриншоты, демонстрирующие запросы к разным фронтендам и ответам от разных бэкендов.


------

### Правила приема работы

1. Необходимо следовать инструкции по выполнению домашнего задания, используя для оформления репозиторий Github
2. В ответе необходимо прикладывать требуемые материалы - скриншоты, конфигурационные файлы, скрипты. Необходимые материалы для получения зачета указаны в каждом задании.


------

### Критерии оценки

- Зачет - выполнены все задания, ответы даны в развернутой форме, приложены требуемые скриншоты, конфигурационные файлы, скрипты. В выполненных заданиях нет противоречий и нарушения логики
- На доработку - задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки, приложены не все требуемые материалы.
