# Физический уровень PostgreSQL 

>cоздайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a

Создал виртуальную машину с Ubuntu 20.04 LTS в Яндекс облаке

>поставьте на нее PostgreSQL 14 через sudo apt

Уставновил PostgreSQL 14:
```sql
ibuyakov@pg:~$ sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $
(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/
keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-14
```
>проверьте что кластер запущен через sudo -u postgres pg_lsclusters

Проверил:
```sql
ibuyakov@pg:~$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

>зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым

Сделано:
```sql
ibuyakov@pg:~$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# create table
postgres-# test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
```
>остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop

Остановил:
```
ibuyakov@pg:~$ sudo -u postgres pg_ctlcluster 14 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@14-main
ibuyakov@pg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```

>создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
>добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
>проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, 
>в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux

Сделано: при создании ВМ я сразу указал необходимость в двух дисках

>сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/

Сдедано:
```
ibuyakov@pg:/var/lib/postgresql/14$ sudo chown -R postgres:postgres /mnt/data/
```

>перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data

Сделано:
```
ibuyakov@pg:/var/lib/postgresql/14$ sudo mv /var/lib/postgresql/14 /mnt/data
```
>попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
>напишите получилось или нет и почему

Сделано:
```
ibuyakov@pg:/var/lib/postgresql/14$ sudo -u postgres pg_ctlcluster 14 main start
Error: /var/lib/postgresql/14/main is not accessible or does not exist
```
Не получилось, потому что в конфигруационном файле указан старый маршрут хранения данных

>задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
>напишите что и почему поменяли

```
ibuyakov@pg:/mnt/data$ sudo nano /etc/postgresql/14/main/postgresql.conf
```
Поменял парамер data_directory
data_directory = '/mnt/data/main'               # use data in another directory

>попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
>напишите получилось или нет и почему

Кластер успешно стартовал
```
ibuyakov@pg:/mnt/data$ sudo -u postgres pg_ctlcluster 14 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@14-main
ibuyakov@pg:/mnt/data$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory Log file
14  main    5432 online postgres /mnt/data/main /var/log/postgresql/postgresql-14-main.log
```

>зайдите через через psql и проверьте содержимое ранее созданной таблицы

```
ibuyakov@pg:/mnt/data$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# select * from test;
 c1
----
 1
(1 row)
```
>задание со звездочкой *: не удаляя существующий GCE инстанс сделайте новый, поставьте на его PostgreSQL, 
>удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй 
>и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.
