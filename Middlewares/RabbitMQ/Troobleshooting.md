RabbitMQ - TroobleShooting
==
<br/>
Quelques commandes permettant d avoir un maximun d informations sur RabbitMQ, la remise en place d'un network partition et l'ajout de vhost dédiés.


#### Network Partitions


```bash
  //remise d'aplomb des nodes d'un cluster rabbit
  rabbitmqctl stop_app
  rabbitmqctl force_reset
  rabbitmqctl join_cluster rabbit@hostname
  rabbitmqctl start_app
  rabbitmqctl cluster_status

```

#### Mise en place d'un node

```bash

  root@mymachine:/etc/rabbitmq # cat rabbitmq-env.conf
  RABBITMQ_USE_LONGNAME=false
  NODENAME=rabbit
  NODE_IP_ADDRESS=0.0.0.0
  NODE_PORT=5672
  HOME=/var/lib/rabbitmq
  MNESIA_BASE=/var/lib/rabbitmq/mnesia

```
#### Error: {ok,already_member}

Source : https://stackoverflow.com/questions/28299716/auto-reconnect-to-rabbitmq-cluster-after-server-restart

Exemple :

```bash

  rabbitmqctl join_cluster --ram rabbit@master
  Clustering node 'rabbit@slave' with 'rabbit@master' ...
  Error: {ok,already_member}

  rabbitmqctl cluster_status
  Cluster status of node 'rabbit@slave' ...
  [{nodes,[{disc,['rabbit@slave']}]}]

  ## Méthode pour remettre le node en place
  ## Attention bien savoir qui est MASTER et SLAVE

  rabbitmqctl -n rabbit@master forget_cluster_node rabbit@slave
  rabbitmqctl join_cluster --ram rabbit@master

```

#### Création User et vhost associé

```bash

    ## Création du vhost
    root@zombie:/etc/rabbitmq # rabbitmqctl add_vhost /vhost1
    Creating vhost "/vhost1" ...

    ## Création user
    root@zombie:/etc/rabbitmq # rabbitmqctl add_user tagada password
    Creating user "tagada" ...

    ## Association user/vhost
    root@zombie:/etc/rabbitmq # rabbitmqctl set_permissions -p /vhost1 tagada ".*" ".*" ".*"
    Setting permissions for user "tagada" in vhost "/vhost1" ...

    ## Associaiton d'un tag spécifique au user
    root@zombie:/etc/rabbitmq # rabbitmqctl set_user_tags tagada administrator

```
