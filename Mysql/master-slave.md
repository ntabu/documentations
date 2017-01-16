Mysql - Replications master/slave
==
<br/>
Dans le cas d un master/slave
#### Remise en place replication sur master bdd1
```bash
# stop slave sur bdd2
stop slave;

# show status du master bdd1
show master status;
+---------------------+----------+--------------+------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+---------------------+----------+--------------+------------------+
| mysql-bin-53.002685 |  2342460 |              |                  |
+---------------------+----------+--------------+------------------+

# On force la réplication du bdd2 en allant à la position du master
CHANGE MASTER TO MASTER_HOST='x.x.x.x', MASTER_USER='replication_user', MASTER_PASSWORD='motdepasse', MASTER_LOG_FILE='mysql-bin-53.002685', MASTER_LOG_POS=2342460;

# Puis start du slave bdd2
start slave;

# Voir l'état du slave
show slave status \G;

```
