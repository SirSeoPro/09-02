# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки» - Кощеев Иван

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Ответ:

<details>

Создайте два файла, server1.py и server2.py, с содержимым:

```

from http.server import BaseHTTPRequestHandler, HTTPServer

class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello from server 1')

server_address = ('', 8001)
httpd = HTTPServer(server_address, SimpleHTTPRequestHandler)
httpd.serve_forever()

```

и

```

from http.server import BaseHTTPRequestHandler, HTTPServer

class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello from server 2')

server_address = ('', 8002)
httpd = HTTPServer(server_address, SimpleHTTPRequestHandler)
httpd.serve_forever()

```

Запустим серверы на в двух терминалах. Сервера будут работать на 8001 и 8002 порту. 

```

python3 server1.py
python3 server2.py

```

Установим и настроим HAProxy

```
sudo apt install haproxy
sudo nano /etc/haproxy/haproxy.cfg

```

Конфигурация: 

```

global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    tcp
    option  tcplog
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server server1 127.0.0.1:8001 check
    server server2 127.0.0.1:8002 check

```

Перезапустим HAProxy и выполним несколько раз curl localhost, что бы посмотреть, какие порты нам отвечают:

![image1](https://github.com/SirSeoPro/09-02/blob/main/1.png)

</details>

### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

### Ответ:

Запустим аналогичный третий сервер на порту 8003.
Оттредактируем конфигурацию HAProxy:
```
global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http_front
    bind *:80
    acl is_example_local hdr(host) -i example.local
    use_backend http_back if is_example_local

backend http_back
    balance roundrobin
    server server1 127.0.0.1:8001 weight 2 check
    server server2 127.0.0.1:8002 weight 3 check
    server server3 127.0.0.1:8003 weight 4 check
```

В /etc/hosts добавим строку:
```
127.0.0.1 example.local
```

используем curl, что бы убедиться, что происходит балансировка нагрузки (так же обратим внимание, что запросы по ip не обрабатываются):

<details>

![image2](https://github.com/SirSeoPro/09-02/blob/main/2.png)

</details>

