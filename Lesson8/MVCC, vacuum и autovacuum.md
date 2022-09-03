# MVCC, vacuum и autovacuum

>создать GCE инстанс типа e2-medium и standard disk 10GB
>установить на него PostgreSQL 14 с дефолтными настройками

```
Сделано
```

>применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла

```
--применил и в дельнейшим использовал для корректировки параметров:
ibuyakov@postgres:~$ sudo nano /etc/postgresql/14/main/postgresql.conf
```

>запустить pgbench
>дать отработать до конца
>зафиксировать среднее значение tps в последней ⅙ части работы
>а дальше настроить autovacuum максимально эффективно
```
Запустил pgbench, установил базовую линию, после чего пытался улучшить показатель tps (транзакций в секунду).
Удалось увеличить с 493.178070 --> 648.185649

ibuyakov@postgres:~$ sudo -u postgres pgbench -c 10 -j 2 -t 10000 example
pgbench (14.5 (Ubuntu 14.5-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
latency average = 20.277 ms
initial connection time = 16.629 ms
tps = 493.178070 (without initial connection time)

ibuyakov@postgres:~$ sudo nano /etc/postgresql/14/main/postgresql.conf

ibuyakov@postgres:~$ sudo -u postgres pgbench -c 10 -j 2 -t 10000 example
pgbench (14.5 (Ubuntu 14.5-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
latency average = 15.965 ms
initial connection time = 16.253 ms
tps = 626.364535 (without initial connection time)

ibuyakov@postgres:~$ sudo nano /etc/postgresql/14/main/postgresql.conf

ibuyakov@postgres:~$ sudo -u postgres pgbench -c 10 -j 2 -t 10000 example
pgbench (14.5 (Ubuntu 14.5-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
latency average = 15.467 ms
initial connection time = 18.562 ms
tps = 646.532210 (without initial connection time)

ibuyakov@postgres:~$ sudo nano /etc/postgresql/14/main/postgresql.conf

ibuyakov@postgres:~$ sudo -u postgres pgbench -c 10 -j 2 -t 10000 example
pgbench (14.5 (Ubuntu 14.5-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
latency average = 15.703 ms
initial connection time = 17.010 ms
tps = 636.805713 (without initial connection time)

ibuyakov@postgres:~$ sudo nano /etc/postgresql/14/main/postgresql.conf

ibuyakov@postgres:~$ sudo -u postgres pgbench -c 10 -j 2 -t 10000 example
pgbench (14.5 (Ubuntu 14.5-0ubuntu0.22.04.1))
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
latency average = 15.428 ms
initial connection time = 22.985 ms
tps = 648.185649 (without initial connection time)
```
