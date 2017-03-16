ElasticSearch - TroobleShooting
==
<br/>
Quelques commandes permettant d avoir un maximun d informations sur ES

#### Status du cluster

```bash
curl -XGET 'http://localhost:9200/_cluster/health?pretty'
{
  "cluster_name" : "twd",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 5,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0
}

```
</br>
#### Status des nodes

```bash
curl -XGET 'http://localhost:9200/_cat/nodes'
rickweb02 192.168.0.3 23 43 1.05 d m rickweb02
carlweb01 192.168.0.2 10 51 1.09 d * carlweb01
```
</br>
#### Etat des index

```bash
curl -XGET 'localhost:9200/_cat/indices?v'
health status index                             pri rep docs.count docs.deleted store.size pri.store.size
green  open   sf_zombi_2016-04-12-200732   5   1    1258898       166350     20.7gb         10.3gb
```
</br>
#### Status des shards

```bash
curl -XGET http://localhost:9200/_cat/shards
sf_zombi_2016-04-12-200732 4 r STARTED 251532   2gb 192.168.0.2 srvweb01
sf_zombi_2016-04-12-200732 4 p STARTED 251532   2gb 192.168.0.3 srvweb02
```
</br>
#### Cluster RED

Généralement, quand un cluster ES est red, c'est peut être dû à un shard non assigné 'unassigned_shards' ou un shard en cours d'initialisaiton 'initializing_shards'.

```bash
  "initializing_shards" : 1,
  "unassigned_shards" : 2,
```

Si le cluster ne se remonte pas automatiquement, forcer le réassignement. (vérifier si l'espace Disk n'est pas plein avant)

```bash

curl -XPOST 'localhost:9200/_cluster/reroute' -d '{
        "commands" : [ {
              "allocate" : {
                  "index" : "<index>",
                  "shard" : 1,
                  "node" : "<noeud>",
                  "allow_primary" : true
              }
            }
        ]
    }';
```

#### Rolling restart

```bash
#arrêt de l'allocation des shards sur le node
curl -XPUT http://localhost:9200/_cluster/settings -d '{"transient":{"cluster.routing.allocation.enable":"none"}}'

#arrêt du node
service elasticsearch restart

#réassignement des shards sur le node
curl -XPUT http://localhost:9200/_cluster/settings -d '{"transient":{"cluster.routing.allocation.enable":"all"}}'

#vérification de l'état du cluster
curl http://localhost:9200/_cluster/'health?pretty'

```
