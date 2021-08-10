 - on slave: stop replication.
```
 stop slave;
 change master to master_auto_position=0,master_log_file='xxxxfile',master_log_pos=xxx;
 start slave;
```
- set all instances' `gtid_mode` = `ON_PERMISSIVE`
```
 - on master: set global GTID_MODE = ON_PERMISSIVE;
 - on slave:  set global GTID_MODE = ON_PERMISSIVE;
``` 
- set all instances' `gtid_mode` = `ON_PERMISSIVE`
```
set global GTID_MODE = OFF_PERMISSIVE;
set global GTID_MODE = OFF_PERMISSIVE;
```

- check all instances' `GLOBAL.GTID_OWNED`, wait until it becomes null.
```
SELECT @@GLOBAL.GTID_OWNED;
```
- check if slave has read all relay log
- set all instances' `gtid_mode` = `off`
```
SET global GTID_MODE = OFF;
SET global ENFORCE_GTID_CONSISTENCY = OFF;
```
- if you want to turn off gtid forever, remenber to delete or comment out gtid related parameters.


[reference]

https://dev.mysql.com/doc/refman/5.7/en/replication-mode-change-online-enable-gtids.html
https://dev.mysql.com/doc/refman/5.7/en/replication-mode-change-online-disable-gtids.html
