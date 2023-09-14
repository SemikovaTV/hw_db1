# Домашнее задание к занятию 2. «SQL» - Семикова Т.В. FOPS-9

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

```bash
  GNU nano 5.4                          docker-compose.yml
version: "3.3"
services:
   semikovatv-db:
    image: postgres:12
    container_name: semikovatv-db
    ports:
     - 5432:5432
    volumes:
      - ./pg_data:/var/lib/postgresql/data/pgdata
      - ./vol1:/var/lib/vol1
      - ./vol2:/var/lib/vol2
    environment:
      POSTGRES_PASSWORD: stv12!3!!
      POSTGRES_DB: semikovatv-netology-db
      PGDATA: /var/lib/postgresql/data/pgdata

    networks:
      semikovatv-hw:
        ipv4_address: 172.1.0.106
    restart: always

networks:
  semikovatv-hw:
    driver: bridge
    ipam:
      config:
      - subnet: 172.1.0.0/24
```

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
```bash
test_db-# \l
                                       List of databases
          Name          |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
------------------------+----------+----------+------------+------------+-----------------------
 postgres               | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 semikovatv-netology-db | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0              | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                        |          |          |            |            | postgres=CTc/postgres
 template1              | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                        |          |          |            |            | postgres=CTc/postgres
 test_db                | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres         +
                        |          |          |            |            | postgres=CTc/postgres
(5 rows)
```  
- описание таблиц (describe);
```bash
test_db-# \d orders
                            Table "public.orders"
 Column |  Type   | Collation | Nullable |              Default
--------+---------+-----------+----------+------------------------------------
 id     | integer |           | not null | nextval('orders_id_seq'::regclass)
 name   | text    |           |          |
 price  | integer |           |          |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_zakaz_fkey" FOREIGN KEY (zakaz) REFERENCES orders(id)

test_db-# \d clients
                                     Table "public.clients"
  Column   |         Type          | Collation | Nullable |               Default
-----------+-----------------------+-----------+----------+-------------------------------------
 id        | integer               |           | not null | nextval('clients_id_seq'::regclass)
 last_name | character varying(30) |           |          |
 country   | character varying(30) |           |          |
 zakaz     | integer               |           |          |
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "clients_zakaz_fkey" FOREIGN KEY (zakaz) REFERENCES orders(id)
```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.
```bash
test_db-# \du
                                       List of roles
    Role name     |                         Attributes                         | Member of
------------------+------------------------------------------------------------+-----------
 postgres         | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test-admin-user  | Superuser, No inheritance                                  | {}
 test-simple-user | No inheritance                                             | {}

test_db=# \z
                                          Access privileges
 Schema |      Name      |   Type   |        Access privileges         | Column privileges | Policies
--------+----------------+----------+----------------------------------+-------------------+----------
 public | clients        | table    | postgres=arwdDxt/postgres       +|                   |
        |                |          | "test-simple-user"=arwd/postgres |                   |
 public | clients_id_seq | sequence |                                  |                   |
 public | orders         | table    | postgres=arwdDxt/postgres       +|                   |
        |                |          | "test-simple-user"=arwd/postgres |                   |
 public | orders_id_seq  | sequence |                                  |                   |
(4 rows)
```

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

```bash
test_db=# insert into orders(id, name, price) VALUES (1, 'Chocolate', 10), (2, 'Принтер',0), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5
```
Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

```bash
test_db=# insert into clients(id, last_name, country) VALUES (1, 'Иванов Иван Иванович','США'), (2, 'Петров Петр Петрович', 'Канада'), (3, 'Иоган Себастьян Бах', 'Япония'), (4, 'Ронни Джеймс Дио', 'Россия'), (5, 'Ritchie Blackmore', 'Россия');
```

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:
    - запросы,
    - результаты их выполнения.
```bash
test_db=# select count (*) from orders;
 count
-------
     5
(1 row)

test_db=# select count (*) from clients;
 count
-------
     5
(1 row)
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.
```bash
test_db=# update  clients set zakaz = 3 where id = 1;
UPDATE 1
test_db=# update  clients set zakaz = 4 where id = 2;
UPDATE 1
test_db=# update  clients set zakaz = 5 where id = 3;
UPDATE 1
```
Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
```bash
test_db=# select c.last_name, o.name from clients c, orders o where c.zakaz = o.id;
      last_name       |  name
----------------------+---------
 Иванов Иван Иванович | Книга
 Петров Петр Петрович | Монитор
 Иоган Себастьян Бах  | Гитара
(3 rows)
```
 
Подсказка: используйте директиву `UPDATE`.

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).
Приведите получившийся результат и объясните, что значат полученные значения.

```bash
test_db=# explain select c.last_name, o.name from clients c, orders o where c.zakaz = o.id;
                               QUERY PLAN
-------------------------------------------------------------------------
 Hash Join  (cost=37.00..52.30 rows=420 width=110)
   Hash Cond: (c.zakaz = o.id)
   ->  Seq Scan on clients c  (cost=0.00..14.20 rows=420 width=82)
   ->  Hash  (cost=22.00..22.00 rows=1200 width=36)
         ->  Seq Scan on orders o  (cost=0.00..22.00 rows=1200 width=36)
(5 rows)
```
Hash join — это алгоритм объединения двух таблиц в базе данных. Он использует функцию хеширования для связывания строк из двух таблиц на основе общего ключа. В процессе хеш-соединения сначала создается хеш-таблица для одной из таблиц, которая используется для быстрого поиска соответствующей строки в другой таблице. Если в хеш-таблице уже есть значение ключа, то соответствующая строка добавляется в результат.
Hash Cond - Показывает нам какие столбцы наших таблиц связываются для вывода.
Seq Scan on clients - последовательное сканирование таблицы "клиенты".
Seq Scan on orders - последовательное сканирование таблицы "заказы".
На этом этапе происходит определение строк, столбцов и их длины, а также помещение результатов в кэш.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---


