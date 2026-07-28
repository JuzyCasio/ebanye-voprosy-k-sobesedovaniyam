# Сети и HTTP для QA/SDET — полный конспект — часть 12

[← Оглавление](networks.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/networks.md)

Темы 117-122.

## 117. Web security базово

XSS  

Внедрение JavaScript в страницу.  

CSRF  

Атака, когда браузер пользователя отправляет нежелательный запрос на сайт, где он авторизован.  

SQL injection  

Вставка SQL в пользовательский ввод.  

SSRF  

Сервер заставляют сделать запрос туда, куда пользователь напрямую не должен обращаться.  

Для QA достаточно понимать симптомы и базовые проверки, но не уходить в эксплуатацию уязвимостей на чужих системах.  

## 118. CSRF и SameSite

Cookie может автоматически отправляться браузером.  

Для защиты от CSRF используют:  

CSRF token;  
SameSite cookie;  
проверку Origin/Referer.  

Cookie attribute:  

SameSite=Lax  
SameSite=Strict  
SameSite=None; Secure  

## 119. Что такое socket

Socket — программная точка сетевого соединения.  

У TCP-соединения есть пара:  

client_ip:client_port -> server_ip:server_port  

### Пример:

192.168.1.10:52344 -> 93.184.216.34:443  

Клиентский порт обычно временный, ephemeral.  

## 120. Ephemeral ports

### Когда клиент подключается к серверу, ОС выбирает временный локальный порт.

### Пример:

Client: 192.168.1.10:52344  
Server: 93.184.216.34:443  

52344 — ephemeral port.  

## 121. Почему может закончиться количество портов

При большом количестве соединений могут закончиться ephemeral ports.  

Симптомы:  

cannot assign requested address;  
много TIME-WAIT;  
проблемы под нагрузкой.  

Решения обычно на стороне настройки ОС, keep-alive, connection pooling, уменьшения лишних соединений.  

## 122. Практические команды для собеса

IP и интерфейсы  
ip addr  
ip link  
Маршруты  
ip route  
DNS  
dig example.com  
nslookup example.com  
cat /etc/resolv.conf  
Проверка доступности  
ping example.com  
traceroute example.com  
Проверка порта  
nc -vz example.com 443  
telnet example.com 443  
HTTP  
curl -v https://example.com  
curl -i https://example.com/api  
Порты на машине  
ss -tulpen  
lsof -i :8080  
Docker  
docker network ls  
docker network inspect bridge  
docker ps  
docker logs container  
tcpdump  
sudo tcpdump -i eth0 port 443  

