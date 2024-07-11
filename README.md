### OTUS Linux Professional Lesson #30-32 | Subject: Firewalls

#### ЦЕЛЬ: Написать сценарии iptables.

#### ЗАДАЧИ:
1. реализовать knocking port (centralRouter может попасть на ssh inetrRouter через knock скрипт)
2. добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост.
3. запустить nginx на centralServer.
4. пробросить 80й порт на inetRouter2 8080.
5. дефолт в инет оставить через inetRouter.

#### ПОРЯДОК ВЫПОЛНЕНИЯ:

Будем использовать следующую схему:

![image](https://github.com/bonyakevich-e/otus_lp_lesson_30-32_firewalls/assets/114911797/411d43ff-25a6-417a-8221-f5587fd4967e)

#### ЗАДАЧА 1:
_Настроить "knocking port" на роутере inetRouter. Результат: к роутеру inetRouter можно подключиться по ssh только если предварительно последовательно постучаться на порты tcp/8881, tcp/7777, tcp/9991._

Подключаемся к роутеру inetRouter:
```
$ vagrant ssh inetRouter
```
Отключаем ufw:
```
root@inetRouter:~# systemctl stop ufw
root@inetRouter:~# systemctl disable ufw
```
Устанавливаем пакеты для сохранения правил iptables:
```
root@inetRouter:~# apt install netfilter-persistent iptables-persistent
```
Включаем маскарадинг:
```
root@inetRouter:~# iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
```
Настраиваем правила для реализации port knocking:
```
root@inetRouter:~# iptables -N TRAFFIC
root@inetRouter:~# iptables -N SSH-INPUT
root@inetRouter:~# iptables -N SSH-INPUTTWO
root@inetRouter:~# iptables -A TRAFFIC -p icmp --icmp-type any -j ACCEPT
root@inetRouter:~# iptables -A TRAFFIC -m state --state ESTABLISHED,RELATED -j ACCEPT
root@inetRouter:~# iptables -A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 22 -m recent --rcheck --seconds 30 --name SSH2 -j ACCEPT
root@inetRouter:~# iptables -A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH2 --remove -j DROP
root@inetRouter:~# iptables -A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 9991 -m recent --rcheck --name SSH1 -j SSH-INPUTTWO
root@inetRouter:~# iptables -A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH1 --remove -j DROP
root@inetRouter:~# iptables -A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 7777 -m recent --rcheck --name SSH0 -j SSH-INPUT
root@inetRouter:~# iptables -A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH0 --remove -j DROP
root@inetRouter:~# iptables -A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 8881 -m recent --name SSH0 --set -j DROP
root@inetRouter:~# iptables -A SSH-INPUT -m recent --name SSH1 --set -j DROP
root@inetRouter:~# iptables -A SSH-INPUTTWO -m recent --name SSH2 --set -j DROP
root@inetRouter:~# iptables -A INPUT -i eth1 -j TRAFFIC
root@inetRouter:~# iptables -A TRAFFIC -i eth1 -j DROP
```
Сохраняем правила:
```
root@inetRouter:~# netfilter-persistent save
```
Проверяем. Делаем попытку подключиться по ssh с centralRouter на inetRouter:
```
vagrant@centralRouter:~$ ssh root@192.168.255.1
```
Видим что подключится не удается. Пробуем теперь постучаться последовательно на порты tcp/8881, tcp/7777, tcp/9991, и после этого подключиться по ssh:
```
vagrant@centralRouter:~$ nmap -Pn --host-timeout 100 --max-retries 0 -p 8881 192.168.255.1
vagrant@centralRouter:~$ nmap -Pn --host-timeout 100 --max-retries 0 -p 7777 192.168.255.1
vagrant@centralRouter:~$ nmap -Pn --host-timeout 100 --max-retries 0 -p 9991 192.168.255.1

vagrant@centralRouter:~$ ssh root@192.168.255.1
root@192.168.255.1's password: 
```
Подключение по ssh работает. Значит port knocking настроен верно. 
Чтобы не знучать какждый раз вручную (не вводить несколько команд), можно сделать скрипт:
```
#!/bin/bash
HOST=$1
shift
for ARG in "$@"
do
        nmap -Pn --host-timeout 100 --max-retries 0 -p $ARG $HOST
done
```
и запускать перед подключением по ssh `knock HOST PORT1 PORT2 PORTx`. Например:
```
$ knock 192.168.255.1 8881 7777 9991
```
