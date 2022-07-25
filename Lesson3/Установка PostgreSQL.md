# Установка PostgreSQL

>сделать в GCE инстанс с Ubuntu 20.04 или развернуть докер любым удобным способом

Создана ВМ на Яндекс облаке c Ubuntu 20.04.4 LTS:

>поставить на нем Docker Engine

Скачал и поставил Docker, сделал пользователя docker
```
ibuyakov@postgre:~$ curl -fsSL https://get.docker.com -o get-docker.sh
ibuyakov@postgre:~$ sudo sh get-docker.sh
ibuyakov@postgre:~$ sudo usermod -aG docker $USER
```

> сделать каталог /var/lib/postgres

Создал:
```
ibuyakov@postgre:/var/lib$ sudo mkdir postgres
```

>развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres

Развернул экземпляр 14 версии, так как был установлен только 12
```
ibuyakov@postgre:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-14
```
Поднял сеть pg-net, развернуть контейнер с PostgreSQL 14:
```
ibuyakov@postgre:/var/lib/postgres$ sudo docker network create pg-net
ibuyakov@postgre:/var/lib/postgres$ sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgres postgres:14
```
>развернуть контейнер с клиентом postgres
>подключится из контейнера с клиентом к контейнеру с сервером и сделать
таблицу с парой строк
```
ibuyakov@postgre:/var/lib/postgres$ sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=# create table test (field1 int, field2 int);
CREATE TABLE
postgres=# insert into test values(1,1),(1,2);
INSERT 0 2
postgres=#  select field1, field2 from test;
 field1 | field2
--------+--------
      1 |      1
      1 |      2
(2 rows)
```
>подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/места установки докера

Поднял другую виртуалку с названием test2 и подключился:
```
ibuyakov@test2:~$ psql -p 5432 -U postgres -h 51.250.25.54 -d postgres -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=#
```
>удалить контейнер с сервером

Остановил и удалил:
```
ibuyakov@postgre:~$ sudo docker stop /pg-docker
/pg-docker
ibuyakov@postgre:~$ sudo docker rm /pg-docker
/pg-docker
```

> создать его заново

Создал:
```
ibuyakov@postgre:~$  sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgres postgres:14
```
>подключится снова из контейнера с клиентом к контейнеру с сервером

Сделано:
```
ibuyakov@postgre:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.
```

>проверить, что данные остались на месте

Данные не остаются на месте, не понимаю в чем проблема. Таблицы test нет после поднятия контейнера.
