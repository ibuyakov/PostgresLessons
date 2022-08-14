# Логический уровень PostgreSQL

>1 создайте новый кластер PostgresSQL 13 (на выбор - GCE, CloudSQL)

```
ibuyakov@postgres:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-14
```
>2 зайдите в созданный кластер под пользователем postgres

```
ibuyakov@postgres:~$ sudo -u postgres psql
```
>3 создайте новую базу данных testdb
```
postgres=# create database testdb;
CREATE DATABASE
```
>4 зайдите в созданную базу данных под пользователем postgres

```
уже под ним
```

>5 создайте новую схему testnm

```
testdb=# create schema testnm;
CREATE SCHEMA
```
>6 создайте новую таблицу t1 с одной колонкой c1 типа integer

```
testdb=# CREATE TABLE t1(c1 integer);
CREATE TABLE
```
>7 вставьте строку со значением c1=1

```
testdb=#  INSERT INTO t1 values(1);
INSERT 0 1
```
>8 создайте новую роль readonly

```
testdb=# create role readonly;
CREATE ROLE
```
>9 дайте новой роли право на подключение к базе данных testdb

```
testdb=# grant connect on DATABASE testdb TO readonly;
```
>10 дайте новой роли право на использование схемы testnm

```
testdb=# grant usage on SCHEMA testnm to readonly;
GRANT
```
>11 дайте новой роли право на select для всех таблиц схемы testnm

```
testdb=# grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
GRANT
```
>12 создайте пользователя testread с паролем test123

```
testdb=# CREATE USER testread with password 'test123';
CREATE ROLE
```
>13 дайте роль readonly пользователю testread

```
testdb=# grant readonly TO testread;
GRANT ROLE
```
>14 зайдите под пользователем testread в базу данных testdb

Зашел, но не сразу. Изначально ловил ошибку:
```
connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "testread"
```
Решил ее, поменяв метод с peer на md5 в pg_hba.conf

```
postgres=# \c testdb testread
Password for user testread:
You are now connected to database "testdb" as user "testread".
testdb=>
```
>15 сделайте select * from t1;

```
testdb=> select * from t1;
ERROR:  permission denied for table t1
```
>16 получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
>17 напишите что именно произошло в тексте домашнего задания
>18 у вас есть идеи почему? ведь права то дали?
>19 посмотрите на список таблиц
>20 подсказка в шпаргалке под пунктом 20
>21 а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)

Таблица была создана на схеме public, а у роли доступ только к таблицам на сзхеме testnm. Прочитал  шпаргалке про **search_path** 
```
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```
>22 вернитесь в базу данных testdb под пользователем postgres

```
testdb=> \c testdb postgres
You are now connected to database "testdb" as user "postgres".
```
>23 удалите таблицу t1

```
testdb=# drop TABLE t1;
DROP TABLE
```

>24 создайте ее заново но уже с явным указанием имени схемы testnm

```
testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE
```

>25 вставьте строку со значением c1=1

```
testdb=# INSERT INTO testnm.t1 values(1);
INSERT 0 1
```
>26 зайдите под пользователем testread в базу данных testdb

```
testdb=# \c testdb testread
Password for user testread:
You are now connected to database "testdb" as user "testread".
```
>27 сделайте select * from testnm.t1;
>28 получилось?
>29 есть идеи почему? если нет - смотрите шпаргалку

```
Нет прав, так как таблица t1 пересоздавалась.
```

>30 как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
>31 сделайте select * from testnm.t1;
>32 получилось?
>33 ура!

```
testdb=> ALTER default privileges in SCHEMA testnm grant SELECT on TABLEs to readonly;
ALTER DEFAULT PRIVILEGES
--затем пересоздал таблицу
testdb=# drop TABLE testnm.t1;
DROP TABLE
testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE
testdb=# INSERT INTO testnm.t1 values(1);
INSERT 0 1
--теперь права есть
testdb=# \c testdb testread;
Password for user testread:
You are now connected to database "testdb" as user "testread".
testdb=> select * from testnm.t1;
 c1
----
  1
(1 row)

```
>34 теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
>35 а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
>36 есть идеи как убрать эти права? если нет - смотрите шпаргалку
>37 если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды

Таблицы создались, но так как мы явно схему не указвали, то они создались на схеме public. А к схеме public есть права у всех пользователей.
Необходимо ограничить права для роли public
```
testdb=# revoke CREATE on SCHEMA public FROM public;
REVOKE
testdb=# revoke all on DATABASE testdb FROM public;
REVOKE
```
>38 теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
>39 расскажите что получилось и почему 

Теперь нет прав на создаине таблиц на схеме public
```
testdb=# \c testdb testread;
Password for user testread:
You are now connected to database "testdb" as user "testread".
testdb=> create table t3(c1 integer); insert into t2 values (2);
ERROR:  permission denied for schema public
LINE 1: create table t3(c1 integer);
```
