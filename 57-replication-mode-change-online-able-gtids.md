
 - master info
```
mysql> show global variables like "%gtid%";
+----------------------------------+----------------------------------------+
| Variable_name                    | Value                                  |
+----------------------------------+----------------------------------------+
| binlog_gtid_simple_recovery      | ON                                     |
| enforce_gtid_consistency         | ON                                     |
| gtid_executed                    | 5eb6574d-9c25-11eb-9608-fa16b833298a:1 |
| gtid_executed_compression_period | 1000                                   |
| gtid_mode                        | OFF                                    |
| gtid_owned                       |                                        |
| gtid_purged                      | 5eb6574d-9c25-11eb-9608-fa16b833298a:1 |
| session_track_gtids              | OFF                                    |
+----------------------------------+----------------------------------------+


mysql> show slave hosts;
+-----------+--------------+-------+-----------+--------------------------------------+
| Server_id | Host         | Port  | Master_id | Slave_UUID                           |
+-----------+--------------+-------+-----------+--------------------------------------+
| 173960025 | 10.94.107.89 | 33066 | 173960470 | 79e5a8f1-9c25-11eb-9c96-fa16f8dfaf30 |
+-----------+--------------+-------+-----------+--------------------------------------+

mysql> select @@global.enforce_gtid_consistency;
+-----------------------------------+
| @@global.enforce_gtid_consistency |
+-----------------------------------+
| ON                                |
+-----------------------------------+
```

 - slave info
```
mysql> show global variables like "%gtid%";
+----------------------------------+----------------------------------------+
| Variable_name                    | Value                                  |
+----------------------------------+----------------------------------------+
| binlog_gtid_simple_recovery      | ON                                     |
| enforce_gtid_consistency         | ON                                     |
| gtid_executed                    | 5eb6574d-9c25-11eb-9608-fa16b833298a:1 |
| gtid_executed_compression_period | 1000                                   |
| gtid_mode                        | OFF                                    |
| gtid_owned                       |                                        |
| gtid_purged                      | 5eb6574d-9c25-11eb-9608-fa16b833298a:1 |
| session_track_gtids              | OFF                                    |
+----------------------------------+----------------------------------------+

mysql> select @@global.enforce_gtid_consistency;
+-----------------------------------+
| @@global.enforce_gtid_consistency |
+-----------------------------------+
| ON                                |
+-----------------------------------+
```

 - Operation:
    - OptID=1与2时 :在master slave 上检查对应的日志中是否有错误信息
    - OptID=4时候，必须保证两边都是zero
    - OptID=5时候，检查步骤4中的binlog在从库上全部应用完，Opt5除非主没有持续写入了 ，可以使用这个检测延迟，操作4完成之后即可开启gtid mode
    
| ID |master|slave|
|:-:|-|-|
|1|SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY=WARN;|SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY=WARN;|
|1|SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY=ON;|SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY=ON;|
|2|SET @@GLOBAL.GTID_MODE = OFF_PERMISSIVE;|SET @@GLOBAL.GTID_MODE = OFF_PERMISSIVE;|
|3|SET @@GLOBAL.GTID_MODE = ON_PERMISSIVE;|SET @@GLOBAL.GTID_MODE = ON_PERMISSIVE;|
|4|SHOW STATUS LIKE 'ONGOING_ANONYMOUS_TRANSACTION_COUNT';|SHOW STATUS LIKE 'ONGOING_ANONYMOUS_TRANSACTION_COUNT';|
|5|show master status;|select MASTER_POS_WAIT('master_log_file',master_log_pos)|
|6|SET @@GLOBAL.GTID_MODE = ON;|SET @@GLOBAL.GTID_MODE = ON;|
|7||STOP SLAVE [FOR CHANNEL 'channel'];
|8||CHANGE MASTER TO MASTER_AUTO_POSITION = 1 [FOR CHANNEL 'channel'];
|9||START SLAVE [FOR CHANNEL 'channel'];|


操作后

 - master
```
mysql> show global variables like "%gtid%";
+----------------------------------+---------------------------------------------+
| Variable_name                    | Value                                       |
+----------------------------------+---------------------------------------------+
| binlog_gtid_simple_recovery      | ON                                          |
| enforce_gtid_consistency         | ON                                          |
| gtid_executed                    | 5eb6574d-9c25-11eb-9608-fa16b833298a:1-3566 |
| gtid_executed_compression_period | 1000                                        |
| gtid_mode                        | ON                                          |
| gtid_owned                       |                                             |
| gtid_purged                      | 5eb6574d-9c25-11eb-9608-fa16b833298a:1      |
| session_track_gtids              | OFF                                         |
+----------------------------------+---------------------------------------------+

```

 - slave
```
mysql>  show global variables like "%gtid%";
+----------------------------------+----------------------------------------------------+
| Variable_name                    | Value                                              |
+----------------------------------+----------------------------------------------------+
| binlog_gtid_simple_recovery      | ON                                                 |
| enforce_gtid_consistency         | ON                                                 |
| gtid_executed                    | 5eb6574d-9c25-11eb-9608-fa16b833298a:1-4545        |
| gtid_executed_compression_period | 1000                                               |
| gtid_mode                        | ON                                                 |
| gtid_owned                       | 5eb6574d-9c25-11eb-9608-fa16b833298a:4546#19021356 |
| gtid_purged                      | 5eb6574d-9c25-11eb-9608-fa16b833298a:1             |
| session_track_gtids              | OFF                                                |
+----------------------------------+----------------------------------------------------+

show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.94.109.22
                  Master_User: slave
                  Master_Port: 33066
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000090
          Read_Master_Log_Pos: 1513918
               Relay_Log_File: relay-log.000002
                Relay_Log_Pos: 829850
        Relay_Master_Log_File: mysql-bin.000090
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 1511986
              Relay_Log_Space: 831983
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
             Master_Server_Id: 173960470
                  Master_UUID: 5eb6574d-9c25-11eb-9608-fa16b833298a
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
           Retrieved_Gtid_Set: 5eb6574d-9c25-11eb-9608-fa16b833298a:3451-4737
            Executed_Gtid_Set: 5eb6574d-9c25-11eb-9608-fa16b833298a:1-4737
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version:
```
