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
Включаем маскарадинг:
```
root@inetRouter:~# iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
```
Устанавливаем пакеты для сохранения правил iptables:
```
root@inetRouter:~# apt install netfilter-persistent iptables-persistent
```
Сохраняем правила:
```
root@inetRouter:~# netfilter-persistent save
```
