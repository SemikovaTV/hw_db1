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
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

---

### Как оформить ДЗ

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

