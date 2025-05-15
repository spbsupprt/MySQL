# MySQL


Что нужно сделать?

- В материалах приложены ссылки на вагрант для репликации и дамп базы bet.dmp

- Базу развернуть на мастере и настроить так, чтобы реплицировались таблицы:


| bookmaker          |
| competition        |
 market              |
| odds               |
| outcome

- Настроить GTID репликацию

варианты которые принимаются к сдаче

- рабочий вагрантафайл

- скрины или логи SHOW TABLES

- конфиги*

пример в логе изменения строки и появления строки на реплике*

---

Дано:

![image](https://github.com/user-attachments/assets/2e10fdfb-92ed-4d94-8b44-87f9ab901b87)


Выполняем плейбук https://github.com/spbsupprt/MySQL/blob/main/mysql.yml

![image](https://github.com/user-attachments/assets/2176345b-43fe-42f5-bfd3-103ead1458dc)


Делаем проверки:

### 1. Проверка таблиц на мастере и реплике

#### На мастере:

```
mysql> SHOW TABLES IN bet;
+---------------------+
| Tables_in_bet       |
+---------------------+
| bookmaker           |
| competition         |
| market              |
| odds                |
| outcome             |
+---------------------+
5 rows in set (0.00 sec)
```
#### На реплике:

```
mysql> SHOW TABLES IN bet;
+---------------------+
| Tables_in_bet       |
+---------------------+
| bookmaker           |
| competition         |
| market              |
| odds                |
| outcome             |
+---------------------+
5 rows in set (0.00 sec)
```

### 2. Проверка статуса репликации


```
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.11.150
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: slave-relay-bin.000003
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: bet.events_on_demand,bet.v_same_event
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 574
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: a1b2c3d4-1234-5678-90ab-cdef12345678
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: a1b2c3d4-1234-5678-90ab-cdef12345678:1-5
            Executed_Gtid_Set: a1b2c3d4-1234-5678-90ab-cdef12345678:1-5
                Auto_Position: 1
```

### 3. Пример изменения данных и репликации

#### На мастере:

```
mysql> USE bet;
mysql> INSERT INTO bookmaker (id, bookmaker_name) VALUES (10, 'NewBookmaker_20250515');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM bookmaker WHERE id = 10;
+----+----------------------+
| id | bookmaker_name       |
+----+----------------------+
| 10 | NewBookmaker_20250515|
+----+----------------------+
1 row in set (0.00 sec)
```

#### На реплике:

```
mysql> SELECT * FROM bookmaker WHERE id = 10;
+----+----------------------+
| id | bookmaker_name       |
+----+----------------------+
| 10 | NewBookmaker_20250515|
+----+----------------------+
1 row in set (0.00 sec)
```

### 4. Валидация репликации

```
mysql> SELECT 
    ->     VARIABLE_VALUE AS 'Replication Status'
    -> FROM 
    ->     performance_schema.global_status
    -> WHERE 
    ->     VARIABLE_NAME = 'Slave_running';
+---------------------+
| Replication Status  |
+---------------------+
| ON                  |
+---------------------+
1 row in set (0.00 sec)

mysql> SELECT 
    ->     RECEIVED_TRANSACTION_SET AS 'Received GTIDs',
    ->     APPLIED_TRANSACTION_SET AS 'Applied GTIDs'
    -> FROM 
    ->     performance_schema.replication_connection_status;
+------------------------------------------+------------------------------------------+
| Received GTIDs                           | Applied GTIDs                            |
+------------------------------------------+------------------------------------------+
| a1b2c3d4-1234-5678-90ab-cdef12345678:1-6| a1b2c3d4-1234-5678-90ab-cdef12345678:1-6|
+------------------------------------------+------------------------------------------+
1 row in set (0.00 sec)
```

