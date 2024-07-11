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

### ЗАДАЧА 1. Настроить "knocking port" на роутере inetRouter. Результат: к роутеру inetRouter можно подключиться по ssh только если предварительно последовательно постучаться на порты tcp/8881, tcp/7777, tcp/9991.

