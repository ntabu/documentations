Mysql - Replications
==
<br/>
Dans le cas d un master/master
#### Remise en place replication sur master bdd2
```bash
#stop slave sur bdd1
mysql -e "stop slave;"

#arrêt de mysql sur bdd2
service mysql stop	//ou /etc/init.d/mysql stop

#Flush des tables et créations du snapshot
mysql
MySQL [(none)]> FLUSH TABLES WITH READ LOCK; SHOW MASTER STATUS; system lvcreate -s -L 10G -n snap2 /dev/systemvm/var_lib_mysql; UNLOCK TABLES;
Query OK, 0 rows affected (0.00 sec)

#Voir la position du bin
mysql
MySQL [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000780 | 23117087 |              |                  |
+------------------+----------+--------------+------------------+

#Création du répertoire snap pour monter le snapshot
if [ ! -d /mnt/snap ] ; then mkdir /mnt/snap ; fi ; mount /dev/system/snap /mnt/snap

#Suppression des anciens log et bin
rm /var/lib/mysql/master.info ; rm /var/lib/mysql/relay-log.info ; rm /var/log/mysql/mysql-bin.* ; rm /var/lib/mysql/mysqld-relay-bin.*

#Suppression du /var/lib/mysql (attention il est conseiller de faire un dump ou un export des tables)
rm -rf /var/lib/mysql/* ; rsync -a <ip_bdd1>:/mnt/snap/ /var/lib/mysql/ 

#Boot de la bdd2
service mysql start bdd2

#Synchro de la bdd2 afin de récuperer la position du bdd1 lors de l arrêt du bdd2
MySQL [(none)]> CHANGE MASTER TO master_host='<ip_bdd1>', master_port=3306, master_user='replication', master_password='<password>', master_log_file='mysql-bin.000780', master_log_pos=23117087;
Query OK, 0 rows affected (0.01 sec)

#Start du slave bdd2
mysql -e "start slave;"

#Voir la position du bdd2 afin que la bdd1 resynchro correctement
MySQL [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      107 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

#Synchro de bdd1
MySQL [(none)]> CHANGE MASTER TO master_host='<ip_bdd2>', master_port=3306, master_user='replication', master_password='<password>', master_log_file='mysql-bin.000001', master_log_pos=1 ;
Query OK, 0 rows affected (0.01 sec)

#Start du slave bdd1
mysql -e "start slave;"

```

Finish !!!
</br>
