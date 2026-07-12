# Домашнее задание к занятию «Кластеризация и балансировка нагрузки»
# - Ушаков Игорь Юрьевич

### Задание 1
```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

# Задание 2: HTTP, Weighted Round Robin, ACL по домену example.local
frontend http_front
    bind :8088
    mode http
    acl is_example_local hdr(host) -i example.local
    use_backend weighted_servers if is_example_local
    default_backend no_service

backend weighted_servers
    mode http
    balance roundrobin
    option httpchk
    http-check send meth GET uri /
    server s1 127.0.0.1:8001 weight=2 check
    server s2 127.0.0.1:8002 weight=3 check
    server s3 127.0.0.1:8003 weight=4 check

backend no_service
    mode http
    # пустой бэкенд — будет возвращать 503
....
```
![задание1](https://github.com/poproshe/zabbix-1/blob/main/img/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-07-12%20160928.png)

### Задание 2

![задание2-3](https://github.com/poproshe/zabbix-1/blob/main/img/img2.png)

