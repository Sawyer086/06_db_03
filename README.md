# Домашнее задание к занятию 3. «MySQL»
## Введение
Перед выполнением задания вы можете ознакомиться с [дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).
## Задача 1
Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с этим контейнером.

## Задача 2
Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней
- количество попыток авторизации — 3
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
  - Фамилия "Pretty"    
  -  Имя "James".
Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.

Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и **приведите в ответе к задаче**.

## Задача 3
Установите профилирование `SET profiling = 1`. Изучите вывод профилирования команд `SHOW PROFILES;`.

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

# Ответ:
## Задача 1:

Команды:
```sql
sudo docker run --name mysql8 -v "/var/docker/mysql9/db:/var/lib/mysql" -v "/var/docker/mysql8:/tmp/backup" -e MYSQL_ALLOW_EMPTY_PASSWORD=yes --rm -d mysql:8
sudo docker exec -it mysql8 bin/bash
bash-4.4# mysql -e 'create database test_db;'
bash-4.4# mysql test_db < /tmp/backup/test_dump2.sql
bash-4.4# mysql
mysql> \s
--------------
mysql  Ver 8.0.33 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          12
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.33 MySQL Community Server - GPL
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 17 min 10 sec

Threads: 2  Questions: 42  Slow queries: 0  Opens: 139  Flush tables: 3  Open tables: 57  Queries per second avg: 0.040
--------------
mysql> connect test_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Connection id:    13
Current database: test_db

mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)

mysql> select count(*) from orders where price>300;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)

```

## Задача 2:
```sql
mysql> CREATE USER 'test' IDENTIFIED WITH mysql_native_password BY 'testpass'
    -> WITH MAX_QUERIES_PER_HOUR 100 PASSWORD EXPIRE INTERVAL 180 DAY FAILED_LOGIN_ATTEMPTS 3
    -> ATTRIBUTE '{"surname": "Pretty", "name": "James"}';
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT SELECT on test_db.* TO test;
Query OK, 0 rows affected (0.02 sec)

mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES;
+------------------+-----------+----------------------------------------+
| USER             | HOST      | ATTRIBUTE                              |
+------------------+-----------+----------------------------------------+
| root             | %         | NULL                                   |
| test             | %         | {"name": "James", "surname": "Pretty"} |
| mysql.infoschema | localhost | NULL                                   |
| mysql.session    | localhost | NULL                                   |
| mysql.sys        | localhost | NULL                                   |
| root             | localhost | NULL                                   |
+------------------+-----------+----------------------------------------+
6 rows in set (0.00 sec)
```
## Задача 3:
```sql
mysql> show table status\G
*************************** 1. row ***************************
           Name: orders
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 5
 Avg_row_length: 3276
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 6
    Create_time: 2024-05-22 20:33:46
    Update_time: 2024-05-22 20:33:47
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.02 sec)

mysql> ALTER TABLE orders ENGINE=MyISAM;
Query OK, 5 rows affected (0.10 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> show table status\G
*************************** 1. row ***************************
           Name: orders
         Engine: MyISAM
        Version: 10
     Row_format: Dynamic
           Rows: 5
 Avg_row_length: 3276
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: 6
    Create_time: 2024-05-22 20:48:07
    Update_time: 2024-05-22 20:33:47
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)

mysql> SHOW PROFILES;
+----------+------------+--------------------------------------+
| Query_ID | Duration   | Query                                |
+----------+------------+--------------------------------------+
|        1 | 0.01112150 | show table status                    |
|        2 | 0.00181500 | ALTER TABLE table_name ENGINE=MyISAM |
|        3 | 0.11148175 | ALTER TABLE orders ENGINE=MyISAM     |
|        4 | 0.00214075 | show table status                    |
+----------+------------+--------------------------------------+
4 rows in set, 1 warning (0.00 sec)

mysql> ALTER TABLE orders ENGINE=InnoDB;
Query OK, 5 rows affected (0.15 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql>
mysql>
mysql>
mysql> SHOW PROFILES;
+----------+------------+--------------------------------------+
| Query_ID | Duration   | Query                                |
+----------+------------+--------------------------------------+
|        1 | 0.01112150 | show table status                    |
|        2 | 0.00181500 | ALTER TABLE table_name ENGINE=MyISAM |
|        3 | 0.11148175 | ALTER TABLE orders ENGINE=MyISAM     |
|        4 | 0.00214075 | show table status                    |
|        5 | 0.16117150 | ALTER TABLE orders ENGINE=InnoDB     |
+----------+------------+--------------------------------------+
5 rows in set, 1 warning (0.00 sec)

## Задача 4:

``` sql
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

#
# * IMPORTANT: Additional settings that can override those from this file!
#   The files must end with '.cnf', otherwise they'll be ignored.
#

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
innodb_log_buffer_size = 1M
innodb_buffer_pool_size = 30
innodb_log_file_size = 100M

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```
