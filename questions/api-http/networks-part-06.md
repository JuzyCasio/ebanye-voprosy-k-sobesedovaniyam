# Сети и HTTP для QA/SDET — полный конспект — часть 6

[← Оглавление](networks.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/networks.md)

Темы 48-60.

## 48. NAT

NAT — Network Address Translation.  

Он позволяет устройствам с приватными IP выходить в интернет через один публичный IP.  

### Пример дома:

Ноутбук: 192.168.1.10  
Телефон: 192.168.1.11  
Роутер: публичный IP  

Для внешнего интернета оба устройства выглядят как один публичный IP.  

### Как объяснить

NAT подменяет адреса источника/назначения в пакетах. Чаще всего используется, чтобы устройства из приватной сети могли выходить в интернет через один публичный IP.  

## 49. Firewall

Firewall фильтрует сетевой трафик.  

Может фильтровать по:  

IP;  
порту;  
протоколу;  
направлению;  
состоянию соединения.  
Примеры  

Разрешить SSH:  

allow tcp/22  

Запретить входящие соединения на порт 5432:  

deny tcp/5432  
Для QA  

Firewall может быть причиной:  

timeout;  
недоступности сервиса;  
недоступности БД;  
проблем между контейнерами;  
ошибок в интеграционных тестах.  

## 50. WAF

WAF — Web Application Firewall.  

Защищает веб-приложение на уровне HTTP.  

Может блокировать:  

SQL injection;  
XSS;  
suspicious payload;  
слишком большие запросы;  
странные headers;  
частые запросы.  
На собесе  

Firewall работает в основном на сетевом/транспортном уровне, а WAF анализирует HTTP-запросы и защищает веб-приложение от атак уровня приложения.  

## 51. VPN

VPN создает защищенный туннель между клиентом и сетью.  

Используется:  

для доступа к корпоративной сети;  
для безопасного соединения;  
для доступа к внутренним ресурсам.  
Частые проблемы QA  
без VPN не доступен стенд;  
DNS внутри VPN отличается;  
маршруты не добавились;  
конфликт локальной и корпоративной подсети;  
сервис доступен по IP, но не по домену.  

## 52. ICMP и ping

ICMP — протокол для служебных сообщений.  

ping использует ICMP Echo Request / Echo Reply.  

ping google.com  
Что проверяет ping  
доступен ли хост на сетевом уровне;  
есть ли ответы;  
примерная задержка.  
Что ping НЕ проверяет  

Ping не проверяет, открыт ли порт приложения.  

Сервер может пинговаться, но HTTP-сервис может не работать.  

## 53. traceroute

Показывает маршрут до хоста.  

traceroute google.com  

На некоторых системах:  

tracepath google.com  

Или Windows:  

tracert google.com  

Используется, чтобы понять, где теряются пакеты.  

## 54. curl

curl — один из главных инструментов QA для проверки HTTP/API.  

GET  
curl http://localhost:8080/users  
Показать headers  
curl -i http://localhost:8080/users  
Только headers  
curl -I http://localhost:8080/users  
POST JSON  
curl -X POST http://localhost:8080/users \  
  -H "Content-Type: application/json" \  
  -d '{"name": "Alex"}'  
Authorization header  
curl http://localhost:8080/users/me \  
  -H "Authorization: Bearer token"  
Подробный вывод  
curl -v https://example.com  
Игнорировать проверку сертификата  
curl -k https://self-signed.local  

Для тестов это иногда полезно, но в production так делать нельзя.  

## 55. telnet / nc

Используются для проверки доступности порта.  

nc  
nc -vz localhost 5432  

### Пример:

Connection to localhost 5432 port [tcp/postgresql] succeeded!  
telnet  
telnet localhost 5432  

Если соединение установилось — порт доступен.  

### На собесе

Ping проверяет доступность хоста, а nc/telnet можно использовать для проверки конкретного TCP-порта.  

## 56. ss, netstat, lsof

ss  

Посмотреть слушающие TCP/UDP порты:  

ss -tulpen  

Где:  

-t — TCP  
-u — UDP  
-l — listening  
-p — process  
-e — extended  
-n — numeric  
netstat  

Старый аналог:  

netstat -tulpen  
lsof  

Какой процесс слушает порт:  

lsof -i :8080  

## 57. ip command

Современная команда для сетевых настроек Linux.  

Посмотреть IP  
ip addr  

### Коротко:

ip a  
Посмотреть маршруты  
ip route  
Посмотреть интерфейсы  
ip link  
Посмотреть ARP/neighbors  
ip neigh  

## 58. ifconfig, route

Старые команды, но иногда встречаются.  

ifconfig  
route -n  

На новых системах лучше:  

ip addr  
ip route  

## 59. Проверка DNS в Linux

nslookup  
nslookup example.com  
dig  
dig example.com  
dig +short example.com  
systemd-resolve / resolvectl  
resolvectl status  
Проверить DNS-серверы  
cat /etc/resolv.conf  

## 60. tcpdump

tcpdump позволяет смотреть сетевой трафик.  

Использовать лучше на своих стендах/тестовых окружениях.  

Посмотреть трафик на интерфейсе  
sudo tcpdump -i eth0  
Фильтр по порту  
sudo tcpdump -i eth0 port 8080  
Фильтр по хосту  
sudo tcpdump -i eth0 host 192.168.1.10  
Сохранить в файл  
sudo tcpdump -i eth0 -w dump.pcap  

Потом файл можно открыть в Wireshark.  

Читать человекопонятно  
sudo tcpdump -i eth0 -nn -vv  

Где:  

-nn — не резолвить имена и порты  
-vv — подробный вывод  

