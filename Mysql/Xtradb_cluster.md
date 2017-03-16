Présentation
------------

### Blabla de présentation "commercial"

Percona XtraDB Cluster est un cluster MySQL actif/actif basé sur
Galera-Cluster. Cette solution associée au proxy HAProxy installé sur
les serveurs applicatifs permet de réaliser une solution de HA globale.

Trois serveurs MySQL sont disponibles en load-balancing par HAProxy avec
désactivation automatique d’un nœud en cas de nécessité (maintenance par
exemple). Plusieurs configurations adaptées aux besoins sont possibles.

L’application accède à un ou plusieurs serveurs MySQL de manière
transparente. La reconstruction d’un nœud est réalisée par Xtrabackup
afin d’assurer une mise à disposition rapide d’un nœud en cas
d’incident.

### Généralités

Percona est un fork de MySQL, Percona XtraDB Cluster est un cluster
MySQL actif/actif basé sur
[1](http://galeracluster.com/products/%7CGalera-cluster) comme MariaDB
Galera Cluster.

Un cluster Galera requière au minimum 2 serveurs mais il est fortement
recommandé d'utiliser au minimum 3 (pour avoir un noeud disponible
pendant la reconstruction d'un noeud HS qui mobilise le 3ième noeud). Il
peut être installé manuellement sur un mysql standard mais Percona
XtraDB Cluster permet de simplifier son installation (exemple : version
module galera correspondant à la version de mysql).

Cette documentation est basée sur debian 7 et Percona XtraDB Cluster
5.6.

Quelques liens pour aller plus loin :

-   <http://www.percona.com/doc/percona-xtradb-cluster/5.6/>
-   <http://galeracluster.com/documentation-webpages/monitoringthecluster.html>
-   <http://www.percona.com/doc/percona-xtradb-cluster/5.6/wsrep-status-index.html#wsrep-status-index>

Quelques informations de bases :

Caractéristiques :

-   Percona Server avec XtraDB utilisant le plugin galera de
    "Synchronous Multi-Master replication"
-   chaque noeud a sa propre copie des données (pas de répartition)
-   bonne solution de répartition de charge de lecture
-   bonne solution de failover
-   le cluster utilise un système de "flow control" qui permet à un
    noeud d'informer le cluster "qu'il est sous l'eau" et de mettre en
    pause la réplication du cluster.

Inconvénients :

-   chaque noeud a sa propre copie des données (il faut donc réaliser
    une copie complète à l'ajout d'un noeud, exemple environ 30-45min
    pour 100Go)
-   éviter tout de même la répartition d'écriture sur tous les noeuds,
    préférez l'utilisation de HAproxy en amont en failover.

Limitations :

-   uniquement InnoDB fonctionne, il n'est pas possible d'avoir des
    tables en MyISAM.
-   Clef primaire obligatoire sur chaque table, les clefs primaires sont
    utilisés dans les phases de certifications des COMMIT.
-   limitation sur la synchronisation de l'écriture (voir détails
    plus bas).
-   pas de support des requêtes de LOCK : LOCK/UNLOCK TABLES et
    fonctions GET\_LOCK(), RELEASE\_LOCK()...
-   logs de requêtes forcément en fichiers, pas vers une table.
-   limitation sur la taille des transactions, les gros traitements
    doivent être revus pour être découpés. Les variables
    wsrep\_max\_ws\_rows et wsrep\_max\_ws\_size définissent
    ces limites.
-   DEADLOCK possible en cas de modification dans une transaction sur
    une même ligne : (Error: 1213 SQLSTATE: 40001 (ER\_LOCK\_DEADLOCK))
-   "XA transactions" non supportées
-   La performance d'écriture sur le cluster est limitée par la
    puissance du noeud le moins performant. Tous les noeuds doivent
    avoir la même puissance pour le pas ralentir les phases
    de certifications.
-   Le cluster doit avoir 3 noeuds minimum (forte recommandation).
-   Le cache de requête n'est plus dynamique et query\_cache\_type doit
    être à 1 au démarrage.

Pour le débug et voir l'état, des variables d'état spécifiques ont été
ajoutées, elles commencent par :

`wsrep_`

Pour les voir :

`show status like "wsrep_%";`

### Quelques détails

Les transactions sont toutes validées sur tous les noeuds au moment du
COMMIT :

![](XtraDBClusterUML1.png "XtraDBClusterUML1.png")

La réplication est synchrone par validation pour réduire la latence des
transactions. Les données sont appliquées de manière asynchrone mais le
noeud valide la "réception" et l'absence de conflit sur la transaction
de manière synchrone :

![](Galera-synchronous-replication.png "Galera-synchronous-replication.png")

Synchro d'un nouveau noeud ou un noeud HS par SST ou IST :

![](Galera-sst-ist.png "Galera-sst-ist.png")

Un SST peut être réalisé par 3 méthodes :

-   mysqldump : lent
-   rsync : rapide
-   Percona Xtrabackup : rapide et non bloquant

IST correspond à un transfert de delta de transaction. La variable
gcache est alors importante pour permettre la réalisation d'un IST en
cas de reboot/maintenance d'un noeud :

`wsrep_provider_options="gcache.size=1G"`

Possibilité de créer des groupes pour la réplication par le WAN :

![](Galera-cluster-wan-replication.jpg "Galera-cluster-wan-replication.jpg")

Dans des cas particuliers, un arbitre peut être ajouté :

![](Galera-cluster-Arbitrator.jpg "Galera-cluster-Arbitrator.jpg")

L'autoincrément est géré automatiquement par répartition des id par
noeuds :

-   Node1
    -   ID : 1,4,7
-   Node2
    -   ID : 2,5,8
-   Node3
    -   ID : 3,6,9

Installation
------------

### Le cluster

#### Global

Lorsque vous avez vos 3 serveurs installés en Debian 7 avec un LAN qui
fonctionne.

Mettre la swappiness à 0 : /etc/sysctl.d/ecritel.conf

`vm.swappiness = 0`

sysctl -p /etc/sysctl.d/ecritel.conf

Récupérer l'archive
![](Mysql percona xtradb cluster scripts.tgz "fig:Mysql percona xtradb cluster scripts.tgz")
et mettre à disposition sur les 3 serveurs MySQL.

Modifier le fichier hosts et ajouter :

`IP1 galera1`\
`IP2 galera2`\
`IP3 galera3`

Ajouter aux sources /etc/apt/sources.list.d/percona.list:

`#Percona`\
`deb `[`http://repo.percona.com/apt`](http://repo.percona.com/apt)` wheezy main`\
`deb-src `[`http://repo.percona.com/apt`](http://repo.percona.com/apt)` wheezy main`

`apt-get update`\
`apt-get install percona-xtradb-cluster-56`

**/!\\ Attention, à partir de la version 2.2.7-5050-1 de
percona-xtrabackup**, dans la configuration ensuite, il faut mettre en
méthode de transfert wsrep\_sst\_method=xtrabackup-v2 au lieu de
wsrep\_sst\_method=xtrabackup.

**Mettre le même mot de passe root sur tous les serveurs.**

Sur un des serveurs, se connecter à MySQL et regardez l'état des
variables :

`mysql> show status like "wsrep_%";`\
`+--------------------------+----------------------+`\
`| Variable_name            | Value                |`\
`+--------------------------+----------------------+`\
`| wsrep_cluster_conf_id    | 18446744073709551615 |`\
`| wsrep_cluster_size       | 0                    |`\
`| wsrep_cluster_state_uuid |                      |`\
`| wsrep_cluster_status     | Disconnected         |`\
`| wsrep_connected          | OFF                  |`\
`| wsrep_local_bf_aborts    | 0                    |`\
`| wsrep_local_index        | 18446744073709551615 |`\
`| wsrep_provider_name      |                      |`\
`| wsrep_provider_vendor    |                      |`\
`| wsrep_provider_version   |                      |`\
`| wsrep_ready              | ON                   |`\
`+--------------------------+----------------------+`\
`11 rows in set (0.00 sec)`

Le cluster n'est pas en fonction.

**Arrêter MySQL sur chaque noeud.**

**Dans my.cnf, pensez à :**

-   Enlever le bind sur localhost
-   Mettre la normalisation pour les caches, voir
    [Normalisation\_des\_configurations\_Linux](Normalisation_des_configurations_Linux "wikilink")

Pour chaque serveur, dans /etc/mysql/my.cnf, section \[mysqld\], ajouter
en adaptant PASSWORD (wsrep\_sst\_auth="sstuser:PASSWORD"), galeraX
(wsrep\_node\_name=galeraX) et IP (wsrep\_node\_address="IP").

Le PASSWORD de sstuser sera utilisé ensuite.

`default_storage_engine=InnoDB`\
`binlog_format=ROW`\
`log_bin=mysql-bin`\
`#Initialise mode`\
`#wsrep_cluster_address=gcomm://`\
`#Active mode`\
`wsrep_cluster_address=gcomm://galera1,galera2,galera3`\
`wsrep_sst_auth="sstuser:PASSWORD"`\
`wsrep_provider=/usr/lib/libgalera_smm.so`\
`wsrep_slave_threads=4`\
`wsrep_cluster_name=galera_cluster`\
`wsrep_node_address="IP"`\
`wsrep_sst_method=xtrabackup-v2`\
`wsrep_node_name=galeraX`\
`#log_slave_updates #If you need to setup replication from an other mysql`\
`innodb_locks_unsafe_for_binlog=1`\
`innodb_autoinc_lock_mode=2`\
`wsrep_provider_options="gcache.size=1G"`\
`#innodb_flush_log_at_trx_commit=0#In case of performance problem`\

**/!\\ Ne pas relancer de suite MySQL**

#### 1er serveur

Nous allons initialiser le cluster, pour cela :

-   Modifier la configuration en décommentant :
    \#wsrep\_cluster\_address=gcomm:// et en commentant
    wsrep\_cluster\_address=gcomm://galera1,galera2,galera3, on a alors
    :

`default_storage_engine=InnoDB`\
`binlog_format=ROW`\
`log_bin=mysql-bin`\
`#Initialise mode`\
`wsrep_cluster_address=gcomm://`\
`#Active mode`\
`#wsrep_cluster_address=gcomm://galera1,galera2,galera3`\
`wsrep_sst_auth="sstuser:PASSWORD"`\
`wsrep_provider=/usr/lib/libgalera_smm.so`\
`wsrep_slave_threads=4`\
`wsrep_cluster_name=galera_cluster`\
`wsrep_node_address="IP"`\
`wsrep_sst_method=xtrabackup-v2`\
`wsrep_node_name=galeraX`\
`log_slave_updates`\
`innodb_locks_unsafe_for_binlog=1`\
`innodb_autoinc_lock_mode=2`\
`wsrep_provider_options="gcache.size=1G"`

-   Puis lancer :

`/etc/init.d/mysql start bootstrap-pxc`

Se connecter à MySQL puis regarder les variables, il y en a alors
beaucoup plus :

`mysql> show status like "wsrep_%";`\
`+------------------------------+--------------------------------------+`\
`| Variable_name                | Value                                |`\
`+------------------------------+--------------------------------------+`\
`| wsrep_local_state_uuid       | 3d138e92-efd7-11e3-b62d-6eb1809e1e63 |`\
`| wsrep_protocol_version       | 5                                    |`\
`| wsrep_last_committed         | 0                                    |`\
`| wsrep_replicated             | 0                                    |`\
`| wsrep_replicated_bytes       | 0                                    |`\
`| wsrep_repl_keys              | 0                                    |`\
`| wsrep_repl_keys_bytes        | 0                                    |`\
`| wsrep_repl_data_bytes        | 0                                    |`\
`| wsrep_repl_other_bytes       | 0                                    |`\
`| wsrep_received               | 2                                    |`\
`| wsrep_received_bytes         | 129                                  |`\
`| wsrep_local_commits          | 0                                    |`\
`| wsrep_local_cert_failures    | 0                                    |`\
`| wsrep_local_replays          | 0                                    |`\
`| wsrep_local_send_queue       | 0                                    |`\
`| wsrep_local_send_queue_avg   | 0.500000                             |`\
`| wsrep_local_recv_queue       | 0                                    |`\
`| wsrep_local_recv_queue_avg   | 0.000000                             |`\
`| wsrep_local_cached_downto    | 18446744073709551615                 |`\
`| wsrep_flow_control_paused_ns | 0                                    |`\
`| wsrep_flow_control_paused    | 0.000000                             |`\
`| wsrep_flow_control_sent      | 0                                    |`\
`| wsrep_flow_control_recv      | 0                                    |`\
`| wsrep_cert_deps_distance     | 0.000000                             |`\
`| wsrep_apply_oooe             | 0.000000                             |`\
`| wsrep_apply_oool             | 0.000000                             |`\
`| wsrep_apply_window           | 0.000000                             |`\
`| wsrep_commit_oooe            | 0.000000                             |`\
`| wsrep_commit_oool            | 0.000000                             |`\
`| wsrep_commit_window          | 0.000000                             |`\
`| wsrep_local_state            | 4                                    |`\
`| wsrep_local_state_comment    | Synced                               |`\
`| wsrep_cert_index_size        | 0                                    |`\
`| wsrep_causal_reads           | 0                                    |`\
`| wsrep_cert_interval          | 0.000000                             |`\
`| wsrep_incoming_addresses     |                                      |`\
`| wsrep_cluster_conf_id        | 1                                    |`\
`| wsrep_cluster_size           | 1                                    |`\
`| wsrep_cluster_state_uuid     | 3d138e92-efd7-11e3-b62d-6eb1809e1e63 |`\
`| wsrep_cluster_status         | Primary                              |`\
`| wsrep_connected              | ON                                   |`\
`| wsrep_local_bf_aborts        | 0                                    |`\
`| wsrep_local_index            | 0                                    |`\
`| wsrep_provider_name          | Galera                               |`\
`| wsrep_provider_vendor        | Codership Oy `<info@codership.com>`    |`\
`| wsrep_provider_version       | 3.5(r178)                            |`\
`| wsrep_ready                  | ON                                   |`\
`+------------------------------+--------------------------------------+`

Celles qui sont importante pour l'instant sont :

`| wsrep_local_state            | 4                                    |`\
`| wsrep_local_state_comment    | Synced                               |`\
`| wsrep_cluster_size           | 1                                    |`\
`| wsrep_cluster_status         | Primary                              |`\
`| wsrep_connected              | ON                                   |`\
`| wsrep_ready                  | ON                                   |`

Si cela ne correspond pas, il y a une erreur.

Se connecter en root puis créer l'utilisateur sstuser ainsi en utilisant
le PASSWORD choisi dans la 1ière étape :

`CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 'PASSWORD';`\
`GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'sstuser'@'localhost';`\
`FLUSH PRIVILEGES;`

#### 2nd et 3ième serveur

Sur le 1ier serveur, donner les droits depuis % à debian-sys-maint :

`GRANT ALL on *.* to 'debian-sys-maint'@'%' identified by 'PASS_DANS_debian.cnf';`

Puis Copier le fichier /etc/mysql/debian.cnf du 1ier serveur sur le 2nd
et 3ième.

Sur le second, relancer MySQL puis contrôler l'état des variables du
cluster.

Le 1ier aura car il y est en cours de transfert vers le nouveau noeud :

`| wsrep_local_state            | 2                                    |`\
`| wsrep_local_state_comment    | Donor/Desynced                       |`

Pendant le transfert, on peut voir des process spécifiques :

`mysql> mysql> show full processlist;`\
`+----+------------------+-----------+------+---------+------+-------------------------+-----------------------+-----------+---------------+`\
`| Id | User             | Host      | db   | Command | Time | State                   | Info                  | Rows_sent | Rows_examined |`\
`+----+------------------+-----------+------+---------+------+-------------------------+-----------------------+-----------+---------------+`\
`|  1 | system user      |           | NULL | Sleep   |  406 | wsrep aborter idle      | NULL                  |         0 |             0 |`\
`|  2 | system user      |           | NULL | Sleep   |    1 | applied write set 26071 | NULL                  |         0 |             0 |`\
`|  3 | system user      |           | NULL | Sleep   |    1 | applied write set 26070 | NULL                  |         0 |             0 |`\
`|  4 | system user      |           | NULL | Sleep   |    0 | applied write set 26072 | NULL                  |         0 |             0 |`\
`|  5 | system user      |           | NULL | Sleep   |    1 | committing 26069        | NULL                  |         0 |             0 |`\
`| 11 | debian-sys-maint | localhost | NULL | Query   |    0 | init                    | show full processlist |         0 |             0 |`\
`+----+------------------+-----------+------+---------+------+-------------------------+-----------------------+-----------+---------------+`\
`6 rows in set (0.00 sec)`

Quand le transfert est terminé, wsrep\_cluster\_size augmente de 1 :

`wsrep_cluster_size           | 2    `

Et l'état revient à 4/Synced :

`| wsrep_local_state            | 4                                    |`\
`| wsrep_local_state_comment    | Synced                               |`

Quand c'est bon pour le second, on refait pareil pour le 3ième et on a :

`| wsrep_local_state            | 4                                                     |`\
`| wsrep_local_state_comment    | Synced                                                |`\
`| wsrep_cluster_size           | 3                                                     |`\
`| wsrep_cluster_status         | Primary                                               |`\
`| wsrep_connected              | ON                                                    |`

Sans activité, les process spécifiques à Galera sont tout de même
présents :

`mysql> show full processlist;`\
`+------+------------------+-----------+------+---------+-------+--------------------+-----------------------+-----------+---------------+`\
`| Id   | User             | Host      | db   | Command | Time  | State              | Info                  | Rows_sent | Rows_examined |`\
`+------+------------------+-----------+------+---------+-------+--------------------+-----------------------+-----------+---------------+`\
`|    1 | system user      |           | NULL | Sleep   |  1576 | committed 84182    | NULL                  |         0 |             0 |`\
`|     2 | system user      |           | NULL | Sleep   | 10756 | wsrep aborter idle | NULL                  |         0 |             0 |`\
`|    3 | system user      |           | NULL | Sleep   |  1576 | committed 84183    | NULL                  |         0 |             0 |`\
`|    4 | system user      |           | NULL | Sleep   |  1576 | committed 84184    | NULL                  |         0 |             0 |`\
`|    5 | system user      |           | NULL | Sleep   |  1576 | committed 84180    | NULL                  |         0 |             0 |`\
`| 9283 | debian-sys-maint | localhost | NULL | Query   |     0 | init               | show full processlist |         0 |             0 |`\
`+------+------------------+-----------+------+---------+-------+--------------------+-----------------------+-----------+---------------+`\
`6 rows in set (0.00 sec)`

#### Retour au 1ier

Tout est bon, on remet la configuration normale sur le 1ier pour my.cnf

`#Initialise mode`\
`#wsrep_cluster_address=gcomm://`\
`#Active mode`\
`wsrep_cluster_address=gcomm://galera1,galera2,galera3`

#### Cache de requête

Vous avez besoin du cache de requête, il faut bien spécifier dans my.cnf
et relancer mysql sinon :

`mysql --defaults-file=/etc/mysql/debian.cnf -e "set global query_cache_type=1;"`\
`ERROR 1651 (HY000) at line 1: Query cache is disabled; restart the server with query_cache_type=1 to enable it`

Dans my.cnf :

`query_cache_type = 1`

### HAProxy et sonde xinetd

Un cluster Percona est généralement utilisé avec HAProxy qui permettra
de répartir les accès vers le cluster.

Haproxy peut être dédié, sur les frontaux WEB, à voir selon la
plateforme. Dans l'exemple, il est installé sur les frontaux.

Haproxy utilisera une sonde sur xinetd pour savoir si le noeud MySQL est
actif.

Sur chaque MySQL :

`apt-get install xinetd`

Créer le fichier /etc/xinetd.d/mysql\_haproxy en complétant avec le LAN
de la plateforme :

`service mysql_haproxy`\
`{`\
`# this is a config for xinetd, place it in /etc/xinetd.d/`\
`        disable = no`\
`        flags           = REUSE`\
`        type            = UNLISTED`\
`        socket_type     = stream`\
`        protocol        = tcp`\
`        port            = 9200`\
`        wait            = no`\
`        user            = root`\
`        server          = /ECRITEL/EXPLOITATION/SCRIPTS/mysql_haproxy_xinetd_check.sh`\
`        log_on_failure  += USERID`\
`        only_from       = 172.16.X.X/XX 213.218.130.0/24`\
`        per_source      = UNLIMITED`\
`}`

Mettre en place
/ECRITEL/EXPLOITATION/SCRIPTS/mysql\_haproxy\_xinetd\_check.sh avec le
fichier présent dans l'archive fournie en début de doc et le rendre
exécutable.

Puis

`sed -i -e "s;disable = no;disable = yes;" /etc/xinetd.d/mysqlchk`

Relancer xinetd :

`/etc/init.d/xinetd restart`

Puis tester le fonctionnement depuis un autre serveur du LAN :

`wget -O - `[`http://172.16.X.X:9200`](http://172.16.X.X:9200)

Doit afficher :

`Percona XtraDB Cluster Node is synced.`

Sur les frontaux, on installe haproxy :

Dans /etc/apt/sources.list.d/backports.list, mettre :

`deb `[`http://cdn.debian.net/debian`](http://cdn.debian.net/debian)` wheezy-backports main`

Puis

`apt-get update`\
`apt-get install haproxy -t wheezy-backports`

Pour la configuration, éditer /etc/haproxy/haproxy.cfg pour avoir comme
ci-dessous et en changeant les IP et le mot de passe :

**Attention à la section global, il faut la conserver.**

`defaults`\
`        log     global`\
`        mode    tcp`\
`        option  tcplog`\
`        option  dontlognull`\
`        timeout connect 5s`\
`        timeout client  1200s`\
`        timeout server  1200s`\
`#       Version < 1.5 :`\
`#        contimeout      5000`\
`#        clitimeout      1200000`\
`#        srvtimeout      1200000`\
`        retries 3`\
`        option redispatch`\
`        maxconn 2000`\
\
`#Default port 3306 with 2 active nodes and 1 node for backups`\
`frontend cluster_default 0.0.0.0:3306`\
`        # detect capacity issues in production farm`\
`        acl cluster_not_enough_capacity nbsrv(cluster_default) lt 2`\
`        # failover traffic to backup farm`\
`        use_backend cluster_all if cluster_not_enough_capacity`\
\
`        default_backend cluster_default`\
\
`frontend cluster_node1 0.0.0.0:3307`\
`        default_backend cluster1`\
\
`frontend cluster_node2 0.0.0.0:3308`\
`        default_backend cluster2`\
\
`frontend cluster_node3 0.0.0.0:3309`\
`        default_backend cluster3`\
\
`frontend cluster_all-nodes 0.0.0.0:3310`\
`        default_backend cluster_all`\
\
\
`backend cluster_default`\
`        mode tcp`\
`        balance leastconn`\
`        option httpchk`\
`        server galera1 IP1:3306 check port 9200 inter 10000 rise 2 fall 3`\
`        server galera2 IP2:3306 check port 9200 inter 10000 rise 2 fall 3`\
`        server galera3 IP3:3306 check port 9200 inter 10000 rise 2 fall 3 backup`\
\
`backend cluster_all`\
`        mode tcp`\
`        balance leastconn`\
`        option httpchk`\
`        server galera1 IP1:3306 check port 9200 inter 10000 rise 2 fall 3`\
`        server galera2 IP2:3306 check port 9200 inter 10000 rise 2 fall 3`\
`        server galera3 IP3:3306 check port 9200 inter 10000 rise 2 fall 3`\
`backend cluster1`\
`        mode tcp`\
`        balance leastconn`\
`        option httpchk`\
`        server galera1 IP1:3306 check port 9200 inter 10000 rise 2 fall 3`\
`        server galera2 IP2:3306 check port 9200 inter 10000 rise 2 fall 3 backup`\
`        server galera3 IP3:3306 check port 9200 inter 10000 rise 2 fall 3 backup`\
`backend cluster2`\
`        mode tcp`\
`        balance leastconn`\
`        option httpchk`\
`        server galera2 IP2:3306 check port 9200 inter 10000 rise 2 fall 3`\
`        server galera1 IP1:3306 check port 9200 inter 10000 rise 2 fall 3 backup`\
`        server galera3 IP3:3306 check port 9200 inter 10000 rise 2 fall 3 backup`\
`backend cluster3`\
`        mode tcp`\
`        balance leastconn`\
`        option httpchk`\
`        server galera3 IP3:3306 check port 9200 inter 10000 rise 2 fall 3`\
`        server galera1 IP1:3306 check port 9200 inter 10000 rise 2 fall 3 backup`\
`        server galera2 IP2:3306 check port 9200 inter 10000 rise 2 fall 3 backup`\
\
`listen admin_page 0.0.0.0:12345`\
`       mode http`\
`       stats uri /`\
`       stats realm Statistiques`\
`       stats auth haproxy:PASSWORD`

Relancer HAProxy puis vérifiez les ports en écoute :

`tcp        0      0 0.0.0.0:12345           0.0.0.0:*               LISTEN      0          338200      -`\
`tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      0          338195      -`\
`tcp        0      0 0.0.0.0:3307            0.0.0.0:*               LISTEN      0          338195      -`\
`tcp        0      0 0.0.0.0:3308            0.0.0.0:*               LISTEN      0          338197      -`\
`tcp        0      0 0.0.0.0:3309            0.0.0.0:*               LISTEN      0          338198      -`\
`tcp        0      0 0.0.0.0:3310            0.0.0.0:*               LISTEN      0          338199      -`

Sur le port 3310, nous sommes en répartition de charge, testons les
accès aux noeuds :

`mysql -u root -p -h 127.0.0.1 -P 3310 --protocol tcp -e 'show variables like "wsrep_node_name";'`

Le résultat change à chaque fois :

`+-----------------+---------+`\
`| Variable_name   | Value   |`\
`+-----------------+---------+`\
`| wsrep_node_name | galera3 |`\
`+-----------------+---------+`

### Monitoring

#### Nagios

Sur les serveurs MySQL, créer le fichier
/etc/nagios/nrpe.d/PERCONAXTRADBCLUSTER.cfg (fichier présent dans
l'archive) :

`command[SERVICE_MYSQL_PERCONAXTRADBCLUSTER-NODES]=/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE_MYSQL_PERCONAXTRADBCLUSTER-NODES -n $ARG1$`\
`command[SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS]=/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS`

Copier les fichiers SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-NODES et
SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS de l'archive pour les mettre
dans /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/

Modifier SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS.conf pour que les
noms des noeuds soient résolus (/etc/hosts) et ajouter d'autres si
nécessaire :

`nodes[0]=galera1`\
`nodes[1]=galera2`\
`nodes[2]=galera3`

Sur WOPR, ajouter les services sur chaque noeud, s'il y en a plus de 3,
modifier l'argument de SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-NODES.

#### Munin

Installer le package ouroboros-exploitation puis mettre dans
/etc/munin/plugin-conf.d/ecritel :

`[mysql_galera_*]`\
`user root`\
`env.mysqlopts --defaults-file=/etc/mysql/debian.cnf`

Faire les liens :

`ln -s /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/munin/mysql_galera_ /etc/munin/plugins/mysql_galera_writesetbytes `\
`ln -s /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/munin/mysql_galera_ /etc/munin/plugins/mysql_galera_writesets`\
`ln -s /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/munin/mysql_galera_ /etc/munin/plugins/mysql_galera_errors`\
`ln -s /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/munin/mysql_galera_ /etc/munin/plugins/mysql_galera_queues`\
`ln -s /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/munin/mysql_galera_ /etc/munin/plugins/mysql_galera_flowcontrol`\
`ln -s /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/munin/mysql_galera_ /etc/munin/plugins/mysql_galera_distance`

Relancer munin-node :

`/etc/init.d/munin-node restart`

### Installation avec ansible

Il s'agit d'un exemple à adapter selon les besoins, basé sur une
installation de la version 5.6 pour Debian 7.

Cette partie considère que vous connaissez déjà l'installation manuelle.

Récupérer le fichier d'archive :
![](Ansible Percona XtraDB Cluster.tgz "fig:Ansible Percona XtraDB Cluster.tgz")
Il contient :

-   roles/percona-xtradb-cluster/tasks/main.yml
-   roles/percona-xtradb-cluster/handlers/main.yml
-   roles/percona-xtradb-cluster/templates/my.cnf
-   roles/percona-xtradb-cluster/templates/debian.cnf
-   roles/percona-xtradb-cluster/templates/xinetd.d/mysql\_haproxy
-   roles/percona-xtradb-cluster/templates/xinetd.d/mysqlchk
-   roles/percona-xtradb-cluster/files/munin\_galera
-   roles/percona-xtradb-cluster/files/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-NODES
-   roles/percona-xtradb-cluster/files/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS
-   roles/percona-xtradb-cluster/files/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS.conf
-   roles/percona-xtradb-cluster/files/PERCONAXTRADBCLUSTER.cfg
-   roles/percona-xtradb-cluster/files/mysql\_haproxy\_xinetd\_check.sh
-   hosts
-   keep-percona-xtradb-cluster-uptodate.yml

Celui-ci ne réalise pas l'installation du package et la création des
utilisateurs. Il gère : les prérequis, les fichiers de configurations et
le monitoring.

On considère que la partie ouroboros est faite dans un role "system" par
exemple.

-   Commencer par modifier hosts et
    keep-percona-xtradb-cluster-uptodate.yml avec le groupe et noms de
    serveurs souhaités. Attention aux enregistrements de /etc/hosts
    nécessaires à MySQL
    et SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS.conf.

<!-- -->

-   Configurer le dépot percona et les prérequis avec :

`ansible-playbook /etc/ansible/keep-percona-xtradb-cluster-uptodate.yml -t percona-base`

-   Sur les serveurs, installer percona et mettre un mot de passe root :

`apt-get update;apt-get install percona-xtradb-cluster-56`\
`/etc/init.d/mysql stop`

-   Sur le serveur ansible, modifier
    **roles/percona-xtradb-cluster/templates/debian.cnf** avec le mot de
    passe du 1er serveur.
-   Sur le serveur ansible, si nécessaire, modifier
    **roles/percona-xtradb-cluster/templates/my.cnf**
    -   avec la configuration des caches
    -   PASSWORD du sstuser
    -   wsrep\_cluster\_address si vous avez changé le nom des serveurs
    -   wsrep\_node\_address si le LAN n'est pas sur eth1 :

wsrep\_node\_address={{ ansible\_eth1.ipv4.address }}
\*\*wsrep\_node\_name si vous souhaitez une valeur particulière, par
défaut :

{{ ansible\_hostname }} \* On déploie :

`ansible-playbook /etc/ansible/keep-percona-xtradb-cluster-uptodate.yml`

-   Ensuite, sur le 1er

`vi /etc/mysql/my.cnf`

Pour initialiser :

`#Initialise mode`\
`wsrep_cluster_address=gcomm://`\
`#Active mode`\
`#wsrep_cluster_address=gcomm://galera1,galera2,galera3`

`/etc/init.d/mysql start bootstrap-pxc`

-   Se connecter à mysql sur le 1er serveur puis créer les droits :
    -   User sst

`CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 'PASSWORD';`\
`GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'sstuser'@'localhost';`\
`FLUSH PRIVILEGES;`

-   -   User debian

`GRANT ALL on *.* to 'debian-sys-maint'@'%' identified by 'PASS_DANS_debian.cnf';`

-   Puis sur les 2 autres, un à un pour les connecter :

`/etc/init.d/mysql start`

-   Enfin on revient au 1er et on remet le mode normal :

`vi /etc/mysql/my.cnf`

Pour remettre en mode actif :

`#Initialise mode`\
`#wsrep_cluster_address=gcomm://`\
`#Active mode`\
`wsrep_cluster_address=gcomm://galera1,galera2,galera3`

Normalement c'est fini.

Il reste la partie HAProxy à mettre dans un autre role lié à votre
architecture.

Exploitation
------------

### Maintenance des vm, Reboot, upgrade version etc...

Il n'y a aucun problème à réaliser des maintenances sur le cluster si le
client utilise bien HAproxy, il y aura bascule entre les noeuds.

Cependant, il faut s'assurer de le faire 1 noeud à la fois puis
contrôler que tout va bien après reboot avec
/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS
-i avant de passer au suivant.

Si vous arrêtez tous les serveurs en même temps, vous allez vous
retrouver dans le cas [cas
5](MySQL_Percona_XtraDB_Cluster#cas_5 "wikilink") ci-dessous.

### Backup

**/!\\ NE PAS LANCER DE DUMP en journée dans le cas d'un cluster de ce
type, cela locke la prévalidation !**

### Erreurs rencontrées

#### Got error 5 during COMMIT

Erreur :

`2014-10-10 14:21:19 28084 [ERROR] Slave SQL: Error in Xid_log_event: Commit could not be completed, 'Got error 5 during COMMIT', Error_code: 1180`\
`2014-10-10 14:21:19 28084 [Warning] Slave: Got error 5 during COMMIT Error_code: 1180`\
`2014-10-10 14:21:19 28084 [ERROR] Error running query, slave SQL thread aborted. Fix the problem, and restart the slave SQL thread with "SLAVE START". We stopped at log 'mysql-bin.009547' position 44917197`

Le problème vient de la limitation sur la taille des transactions
(taille ou lignes, dans l'exemple nous avions eu sur la taille). Par
défaut wsrep\_max\_ws\_size vaut 1G. On le passe à 2G et le client doit
changer son traitement pour éviter les problèmes.

wsrep\_max\_ws\_size=2G dans my.cnf et application en live :

`mysql --defaults-file=/etc/mysql/debian.cnf -e "set global wsrep_max_ws_size=2048*1024*1024;show variables like 'wsrep_max_ws_size';"`\
`+-------------------+------------+`\
`| Variable_name     | Value      |`\
`+-------------------+------------+`\
`| wsrep_max_ws_size | 2147483648 |`\
`+-------------------+------------+`

Dans le cas d'une réplication, relancer l'esclave. La requête sera très
longue à s'exécuter.

#### WSREP: SST failed: 32 (Broken pipe)

Si par exemple après un upgrade de version, vous avez cette erreur dans
les logs /var/lib/mysql/SERVER.err :

`2015-01-06 10:41:37 2591 [ERROR] WSREP: Process completed with error: wsrep_sst_xtrabackup --role 'joiner' --address '172.17.0.6' --auth 'sstuser:PASSWORD' --datadir '/var/lib/mysql/' --defaults-file '/etc/mysql/my.cnf' --parent '2591'  '' : 32 (Broken pipe)`\
`2015-01-06 10:41:37 2591 [ERROR] WSREP: Failed to read uuid:seqno from joiner script.`\
`2015-01-06 10:41:37 2591 [ERROR] WSREP: SST failed: 32 (Broken pipe)`\
`2015-01-06 10:41:37 2591 [ERROR] Aborting`

Attention, à partir de la version 2.2.7-5050-1 de percona-xtrabackup,
dans la configuration ensuite, il faut mettre en méthode de transfert
wsrep\_sst\_method=xtrabackup-v2 au lieu de
wsrep\_sst\_method=xtrabackup.

Modifier la configuration en wsrep\_sst\_method=xtrabackup-v2, puis
relancer mysql.

### Monitoring

#### SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-NODES

-   Systèmes concernés : Systèmes Linux (Si c'est sur windows, bah caca)
-   Description : Vérifie que le nombre de noeud défini (3 par défaut,
    plus en argument si nécessaire) correspond au nombre de serveur dans
    le cluster.
-   Source : Nagios

`Normalement, s'il est en alerte, SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS doit l'être aussi, suivre la procédure de SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS.`

#### SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS

-   Systèmes concernés : Systèmes Linux (Si c'est sur windows, bah caca)
-   Description : Vérifie l'état du Percona XtraDB Cluster.
-   Source : Nagios

##### Information sur ce monitoring

Le script est celui-ci
/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS.

Si besoin, vous pouvez lancer le script avec l'option -i pour avoir plus
de détails (peut-être plus pour le N2) :

`/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS -i`

Vous trouverez la liste des serveurs dans le cluster dans le fichier de
configuration du monitoring :
/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS.conf

`nodes[0]=etai125.lan`\
`nodes[1]=etai126.lan`\
`nodes[2]=etai127.lan`

Le monitoring est présent pour tous les noeuds et n'a pas forcément la
même valeur selon le noeud.

##### Information valable pour tous les cas

Ceci est une note d'information commune, suivre les cas comme indiqué.

Il arrive fréquemment que la relance indique une erreur, mais mysql est
bien en cours de démarrage :

`[FAIL] Starting MySQL (Percona XtraDB Cluster) database server: mysqld[....] Please take a look at the syslog. ... failed!`\
`failed!`

Vérifier avec un

`ps faux | grep 'mysql'`\
`root     27195  0.0  0.0   8324   884 pts/1    S+   14:14   0:00          \_ grep mysql`\
`root     18404  0.0  0.0   4180   768 ?        S    14:09   0:00 /bin/sh /usr/bin/mysqld_safe`\
`mysql    20239 99.0 12.6 10593512 2077308 ?    Sl   14:10   4:00  \_ /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plug`

Relancer
/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS
-i et après quelques minutes, il n'y aura plus d'erreur.

Si ce n'est pas le cas, alors il faut bien suivre chaque cas, et appeler
le N2 si besoin.

En cas de relance mysql un peu tardive, MySQL va surement lancer un
transfert complet de la base, la relance sort en erreur et le temps de
synchro peut durer 1 heure selon le volume de données à transferer,
c'est normal.

Exemple :

`root@etai125:/var/lib/mysql# /etc/init.d/mysql restart`\
`[ ok ] Stopping MySQL (Percona XtraDB Cluster): mysqld.`\
`[FAIL] Starting MySQL (Percona XtraDB Cluster) database server: mysqld[....] Please take a look at the syslog. ... failed!`\
`failed!`

Faire un ps auxf :

`root     26273  0.2  0.0   4180   772 pts/2    S    09:02   0:00 /bin/sh /usr/bin/mysqld_safe`\
`mysql    27187  7.1  3.4 2312020 571932 pts/2  Sl   09:03   0:00  \_ /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plugin --user=mysql --wsrep-provid`\
`mysql    27239  0.0  0.0   4180   576 pts/2    S    09:03   0:00      \_ sh -c wsrep_sst_xtrabackup --role 'joiner' --address '172.16.18.56' --auth 'sstuser:fe0jXudtPW' --datadir '/var/lib`\
`mysql    27240  0.0  0.0  11480  1764 pts/2    S    09:03   0:00          \_ /bin/bash -ue /usr//bin/wsrep_sst_xtrabackup --role joiner --address 172.16.18.56 --auth sstuser:fe0jXudtPW --d`\
`mysql    27380  0.0  0.0  21608  1404 pts/2    S    09:03   0:00              \_ socat -u TCP-LISTEN:4444,reuseaddr stdio`\
`mysql    27381  0.0  0.0  10608   648 pts/2    S    09:03   0:00              \_ tar xfi - --recursive-unlink -h`

On voit le process de transfert depuis un autre noeud.

##### 1ière action à réaliser pour tous les cas

Se connecter sur un premier serveur en alerte, puis faire :

`cat /ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS.conf`

Cela vous donne la liste des serveurs du cluster.

Puis il faut commencer par se connecter sur tous les noeuds par ssh puis
lancer le script pour voir l'état actuel (cela évolue vite par exemple
en cas de reboot d'une VM, il faut donc s'assurer de l'état courant).

`/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS`

En fonction de l'état retourné par le script, suivez les instructions
correspondant aux cas décrits ci-dessous :

##### cas 1

La sortie est pour tous du type :

`OK - cluster_status: Primary cluster_state_uuid:3d138e92-efd7-11e3-b62d-6eb1809e1e63`

Le cluster a dû se synchroniser suite à une relance de MySQL ou un autre
problème temporaire. Envoyer à l'exploitation pour suivi sans appel de
l'astreinte pour contrôler la raison de l'indisponibilité du noeud.

##### cas 2

Un des noeuds indique :

`WARNING - sst running, please wait`

La synchronisation est en cours suite à une relance de MySQL ou un
reboot de VM. Les autres noeuds indique un message du type (adapter le
nom du serveur) :

`CRITICAL - is etai126.lan down or in SST mode ?`

Un des noeuds peut avoir le status :

`etai125.lan is Donor`

C'est normal pendant qu'un des autres est en **WARNING - sst running,
please wait**.

Dans ce cas, il faut attendre 15 à 30 minutes puis recontrôler que tout
est repassé à OK. Si au bout de 45m-1h, ce n'est toujours pas à OK,
appeler l'astreinte.

Si tout est revenu à OK, envoyer à l'exploitation pour suivi sans appel
de l'astreinte pour contrôler la raison de l'indisponibilité du noeud.

##### cas 3

Tous les noeuds indiquent :

`CRITICAL - is etai126.lan down or in SST mode ?`

Relancer MySQL sur le noeud indiqué comme down. Ne relancez pas le
mauvais ou il va y avoir une indispo ! c'est long et normal :

`/etc/init.d/mysql start`\
`[ ok ] Starting MySQL (Percona XtraDB Cluster) database server: mysqld . . . . . . . . . . . . . . . . . . . . .[....] State transfer in progress, setting sleep higher: mysqld ..`

Ensuite, relancez le script
/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS
sur tous les serveurs. Vous devez arriver au **cas 2**. Suivre les
instructions du cas 2.

Si ce n'est pas le cas, contacter l'astreinte N2.

##### cas 4

Si l'alerte est :

`CRITICAL - etai126.lan not Connected`

ou

`CRITICAL - etai126.lan not Ready`

Relancez MySQL sur le serveur correspondant. Ne relancez pas le mauvais
ou il va y avoir une indispo !

`/etc/init.d/mysql restart`\
`[ ok ] Starting MySQL (Percona XtraDB Cluster) database server: mysqld . . . . . . . . . . . . . . . . . . . . .[....] State transfer in progress, setting sleep higher: mysqld ..`

Ensuite, relancez le script
/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE\_MYSQL\_PERCONAXTRADBCLUSTER-STATUS
sur tous les serveurs. Vous devez arriver au **cas 2**. Suivre les
instructions du cas 2.

Si ce n'est pas le cas, contacter l'astreinte N2.

##### cas 5

Après relance de tous les noeuds, ou en cas de crash (ESX HS par
exemple), les mysql ne redémarrent pas, on a alors :

`CRITICAL - is etai125.lan down or in SST mode ? - is etai126.lan down or in SST mode ? - is etai127.lan down or in SST mode ?`

En mode débug :

`/ECRITEL/EXPLOITATION/SCRIPTS/monitoring/nagios/SERVICE_MYSQL_PERCONAXTRADBCLUSTER-STATUS -i`\
`ERROR 2003 (HY000): Can't connect to MySQL server on 'etai125.lan' (111)`\
`ERROR 2003 (HY000): Can't connect to MySQL server on 'etai126.lan' (111)`\
`ERROR 2003 (HY000): Can't connect to MySQL server on 'etai127.lan' (111)`\
`Node etai125.lan: down or in SST mode`\
\
`Node etai126.lan: down or in SST mode `\
\
`Node etai127.lan: down or in SST mode`\
\
`CRITICAL - is etai125.lan down or in SST mode ? - is etai126.lan down or in SST mode ? - is etai127.lan down or in SST mode ?`

Alors là, c'est un peu plus le caca, on va appeler l'astreinte N2 pour
la suite ! ET on lui donne la page wiki pour ce cas.

Donc tous les serveurs sont stoppés. Si on a de la chance et que les
serveurs ont été arrêtés proprement alors MySQL écrit dans un fichier,
on va regarder celui-ci : /var/lib/mysql/grastate.dat pour sa date et
son contenu (seqno) :

`ls -l /var/lib/mysql/grastate.dat`

`cat /var/lib/mysql/grastate.dat`\
`# GALERA saved state`\
`version: 2.1`\
`uuid:    c3651f3e-4012-11e4-9a95-322ca79f5709`\
`seqno:   3`\
`cert_index:`

Choisir le noeud qui s'est arrêté en dernier. Puis on le relance :

`/etc/init.d/mysql bootstrap-pxc`

Lorsqu'il est bien démarré, on va démarrer normalement par
/etc/init.d/mysql start, les noeuds restant mais un puis l'autre en
attendant que le SST soit terminé.

On a moins de chance, et les serveurs ont crashés, alors, sur chaque on
regarde avec la commande suivante la **Recovered position** (ce qui est
après le : ), et on choisit le noeud avec la position la plus avancée.

`mysqld_safe --wsrep-recover`\
`140821 15:57:15 mysqld_safe Logging to '/var/lib/mysql/percona3_error.log'.`\
`140821 15:57:15 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql`\
`140821 15:57:15 mysqld_safe WSREP: Running position recovery with --log_error='/var/lib/mysql/wsrep_recovery.6bUIqM' --pid-file='/var/lib/mysql/percona3-recover.pid'`\
`140821 15:57:17 mysqld_safe WSREP: Recovered position 4b83bbe6-28bb-11e4-a885-4fc539d5eb6a:2`\
`140821 15:57:19 mysqld_safe mysqld from pid file /var/lib/mysql/percona3.pid ended`

Puis sur le noeud choisit :

`/etc/init.d/mysql bootstrap-pxc`

Lorsqu'il est bien démarré, on va démarrer normalement par
/etc/init.d/mysql start, les noeuds restant mais un puis l'autre en
attendant que le SST soit terminé.

##### cas XXX ou complément d'informations pour l'astreinte

<http://www.percona.com/blog/2014/09/01/galera-replication-how-to-recover-a-pxc-cluster/>
