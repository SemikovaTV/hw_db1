# Домашнее задание к занятию 3. «MySQL» - Семикова Т.В. FOPS-9

## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.
```bash
version: "3.3"
services:
   mysql-server:
    image: mysql:8.0
    container_name: msql-db
    ports:
     - 1234:1234
    volumes:
      - type: bind
        source: $HOST/home/stv
        target: /home/stv
      - ./dbdata:/var/lib/mysql
      - ./msql_vol1:/var/lib/msql_vol1
      - ./msql_vol22:/var/lib/msql_vol2
    environment:
      MYSQL_DATABASE: test_stv
      MYSQL_ROOT_PASSWORD: stv
      SERVICE_NAME: mysql

    networks:
      semikovatv-hw:
        ipv4_address: 172.1.0.111
    restart: always

networks:
  semikovatv-hw:
    driver: bridge
    ipam:
      config:
      - subnet: 172.1.0.0/24
```

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.
Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

```sql
mysql> \s
--------------
mysql  Ver 8.0.34 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          15
Current database:       test_db
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.34 MySQL Community Server - GPL
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 9 min 42 sec

Threads: 2  Questions: 70  Slow queries: 0  Opens: 192  Flush tables: 3  Open tables: 110  Queries per second avg: 0.120
--------------
```
Подключитесь к восстановленной БД и получите список таблиц из этой БД.
```sql
mysql> USE test_db;
Database changed
mysql> SHOW TABLES;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)
```
**Приведите в ответе** количество записей с `price` > 300.
```sql
mysql> SELECT COUNT(*) FROM orders WHERE price > 300;
+----------+
| COUNT(*) |
+----------+
|        1 |
+----------+
1 row in set (0.02 sec)
```
В следующих заданиях мы будем продолжать работу с этим контейнером.

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".
  
```sql
mysql> CREATE USER 'test' IDENTIFIED BY 'test-pass';
Query OK, 0 rows affected (0.05 sec)
mysql> ALTER USER 'test' ATTRIBUTE '{"fname":"James", "lname":"Pretty"}';
Query OK, 0 rows affected (0.04 sec)
mysql> ALTER USER 'test'
    -> IDENTIFIED BY 'test-pass'
    -> WITH
    -> MAX_QUERIES_PER_HOUR 100
    -> PASSWORD EXPIRE INTERVAL 180 DAY
    -> FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2;
Query OK, 0 rows affected (0.05 sec)
```

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
 ```sql
mysql> GRANT SELECT ON test_db .* TO 'test';
Query OK, 0 rows affected (0.03 sec)
```   
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и **приведите в ответе к задаче**.
```sql
mysql> SELECT * FROM information_schema.user_attributes WHERE USER = 'test';
+------+------+---------------------------------------+
| USER | HOST | ATTRIBUTE                             |
+------+------+---------------------------------------+
| test | %    | {"fname": "James", "lname": "Pretty"} |
+------+------+---------------------------------------+
1 row in set (0.00 sec)
```
## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;
```sql
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> SHOW PROFILES;
+----------+------------+----------------------------------------+
| Query_ID | Duration   | Query                                  |
+----------+------------+----------------------------------------+
|        1 | 0.00090575 | SELECT * FROM orders WHERE price > 100 |
|        2 | 0.00531000 | SELECT * FROM orders WHERE price > 200 |
|        3 | 0.00052200 | SELECT * FROM orders WHERE price > 300 |
+----------+------------+----------------------------------------+
3 rows in set, 1 warning (0.00 sec)
```
Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.
```sql
mysql> SELECT table_schema, table_name, engine FROM information_schema.tables WHERE table_name = 'orders';
+--------------+------------+--------+
| TABLE_SCHEMA | TABLE_NAME | ENGINE |
+--------------+------------+--------+
| test_db      | orders     | InnoDB |
+--------------+------------+--------+
1 row in set (0.00 sec)
```
Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.
```sql
mysql> ALTER TABLE orders ENGINE =MyISAM;
Query OK, 5 rows affected (0.21 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILES;
+----------+------------+----------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                              |
+----------+------------+----------------------------------------------------------------------------------------------------+
|        1 | 0.00090575 | SELECT * FROM orders WHERE price > 100                                                             |
|        2 | 0.00531000 | SELECT * FROM orders WHERE price > 200                                                             |
|        3 | 0.00052200 | SELECT * FROM orders WHERE price > 300                                                             |
|        4 | 0.00849625 | SELECT table_schema, table_name, engine FROM information_schema.tables WHERE table_name = 'orders' |
|        5 | 0.00013700 | ALTER TABLE orders ENGINE =MyIASM                                                                  |
|        6 | 0.21078875 | ALTER TABLE orders ENGINE =MyISAM                                                                  |
|        7 | 0.00056325 | SELECT * FROM orders WHERE price > 100                                                             |
|        8 | 0.00035425 | SELECT * FROM orders WHERE price > 200                                                             |
|        9 | 0.00153250 | SELECT * FROM orders WHERE price > 300                                                             |
+----------+------------+----------------------------------------------------------------------------------------------------+
9 rows in set, 1 warning (0.00 sec)
```
- на `MyISAM`
```sql
7 | 0.00056325 | SELECT * FROM orders WHERE price > 100 
```
- на `InnoDB`
```sql
1 | 0.00090575 | SELECT * FROM orders WHERE price > 100
 ```

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.
```bash
bash-4.4# cat my.cnf
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M

# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

innodb_flush_log_at_trx_commit=0
innodb_file_per_table=ON
innodb_log_buffer_size=1M
innodb_buffer_pool_size=700M
innodb_log_file_size=100M

pid-file=/var/run/mysqld/mysqld.pid
[client]
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/
```
