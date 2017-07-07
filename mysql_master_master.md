## mysql master-master


```
### Env

mysql1: 10.10.56.10
mysql2: 10.10.56.20
```

1. install mariadb

```
### on mysql1

# yum install mariadb mariadb-server
```

```
### on mysql2

# yum install mariadb mariadb-server
```


2. config my.cnf

```
### on mysql1

# vim /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

port = 3306
server_id = 1
log-bin= mysql-bin
relay_log=mysql-relay-bin
binlog_format = row
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
auto-increment-offset=1
auto-increment-increment=2

# systemctl restart mariadb
```

```
### on mysql2

# vim /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

port = 3306
server_id = 2
log-bin= mysql-bin
relay_log=mysql-relay-bin
binlog_format = row
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
auto-increment-offset=2
auto-increment-increment=2

# systemctl restart mariadb
```


3. add replication user on mysql1

```
### on mysql1

# mysql -uroot -ppassword
MariaDB [(none)]> grant replication slave,replication client on *.* to 'repluser'@'10.10.56.%' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges; 
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show master status; 
+------------------+----------+--------------+--------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB         |
+------------------+----------+--------------+--------------------------+
| mysql-bin.000004 |      504 |              | mysql,information_schema |
+------------------+----------+--------------+--------------------------+
1 row in set (0.00 sec)

```


4. config mysql2 master to mysql1

```
### on mysql2

# mysql -uroot -ppassword

MariaDB [(none)]> stop slave;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> change master to MASTER_HOST='10.10.56.10', MASTER_USER='repluser', MASTER_PASSWORD='123456',  MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=504;   
Query OK, 0 rows affected (0.14 sec)

MariaDB [(none)]> start slave;        
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.10.56.20
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 504
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 529
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 

                ... ...

               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
1 row in set (0.00 sec)

```


5. add replication user on mysql2

```
### on mysql2

# mysql -uroot -ppassword
MariaDB [(none)]> grant replication slave,replication client on *.* to 'repluser'@'10.10.56.%' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges; 
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show master status; 
+------------------+----------+--------------+--------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB         |
+------------------+----------+--------------+--------------------------+
| mysql-bin.000004 |      504 |              | mysql,information_schema |
+------------------+----------+--------------+--------------------------+
1 row in set (0.00 sec)

```


6. config mysql1 master to mysql2

```
### on mysql1

# mysql -uroot -ppassword

MariaDB [(none)]> stop slave;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> change master to MASTER_HOST='10.10.56.20', MASTER_USER='repluser', MASTER_PASSWORD='123456',  MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=504;   
Query OK, 0 rows affected (0.14 sec)

MariaDB [(none)]> start slave;        
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.10.56.10
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 504
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 529
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 

                ... ...

               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 2
1 row in set (0.00 sec)

```


7. check 

```
### on mysql1

MariaDB [(none)]> create database testsync;
Query OK, 1 row affected (0.00 sec)

### on mysql2

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| testsync           |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> create database testsync2;
Query OK, 1 row affected (0.00 sec)

### on mysql1
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| testsync           |
| testsync2          |
+--------------------+

```


