# Lesson2
>создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере 
>, например postgres2022-, где yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально на уровне GCP)

**Проект был создан**

>далее создать инстанс виртуальной машины Compute Engine с дефолтными параметрами

**Создана ВМ на Яндекс облаке c Ubuntu 20.04.4 LTS**

>добавить свой ssh ключ в GCE metadata

**В Яндекс облаке публичный ключ SSH задается при создании ВМ. Публичный и приватный ключи были созданы с помощью втроенной в Windows10 утилиты ssh:**
```cmd
C:\Users\buyakov_iliya>ssh-keygen -t rsa -b 2048
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\buyakov_iliya/.ssh/id_rsa): postgretest
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in postgretest.
Your public key has been saved in postgretest.pub.
```

>зайти удаленным ssh (первая сессия), не забывайте про ssh-add

**Сделано:**
```
C:\Users\buyakov_iliya>ssh -i postgretest ibuyakov@51.250.109.237
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-122-generic x86_64)
ibuyakov@postgre:~$
```

>поставить PostgreSQL

**Установил Postgre 12 версии, следующим образом:**
```
ibuyakov@postgre:~$ sudo apt-get -y install postgresql
```

>зайти вторым ssh (вторая сессия)

**Аналогично подключился к ВМ через вторую сессию:**
```
C:\Users\buyakov_iliya>ssh -i postgretest ibuyakov@51.250.109.237
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-122-generic x86_64)
ibuyakov@postgre:~$
```

>запустить везде psql из под пользователя postgres

**Сделано:**
```
ibuyakov@postgre:~$ sudo -u postgres psql
psql (12.11 (Ubuntu 12.11-0ubuntu0.20.04.1))
Type "help" for help.
```
>выключить auto commit

**Сделано:**
```
postgres=# \set AUTOCOMMIT OFF
```

>сделать в первой сессии новую таблицу и наполнить ее данными 
>create table persons(id serial, first_name text, second_name text); 
>insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
>insert into persons(first_name, second_name) values('petr', 'petrov'); 
>commit;

**Сделано:**
```
postgres=#  create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
CREATE TABLE
INSERT 0 1
INSERT 0 1
COMMIT
```

>посмотреть текущий уровень изоляции: show transaction isolation level

**Сделано:**
```
postgres=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```

>начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
>в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
>сделать select * from persons во второй сессии
>видите ли вы новую запись и если да то почему?
>завершить первую транзакцию - commit;
>сделать select * from persons во второй сессии
>видите ли вы новую запись и если да то почему?
>завершите транзакцию во второй сессии

**Ответ:**
Во втором окне мы не увидим запись с first_name = 'sergey' в таблице persons, так как в первом окне мы выключили автокоммит, 
поэтому транзакция со вставкой данных еще осуществляется, а уровень изоляции read committed позволяет читать только закомиченные данные.
И только когда мы сделаем коммит транзакции с инсертом в первом окне, при повторной выборке данных во втором окне мы увидим запись 
с first_name = 'sergey' в таблице persons.


>начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
>в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
>сделать select * from persons во второй сессии
>видите ли вы новую запись и если да то почему?
>завершить первую транзакцию - commit;
>сделать select * from persons во второй сессии
>видите ли вы новую запись и если да то почему?
>завершить вторую транзакцию
>сделать select * from persons во второй сессии

**Ответ:**
Во втором окне мы не увидим запись с first_name = 'sveta' в таблице persons, потому что фантомное чтение при уровне изоляции repeatable read в Postgre не допускается.
После коммитта транзакции со вставкой мы увидим во второй сессии запись с first_name = 'sveta' в таблице persons


