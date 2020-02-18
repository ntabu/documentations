ElasticSearch - TroobleShooting
==
<br/>
Quelques commandes permettant d avoir un maximun d informations sur ES
"?pretty" à la fin d'une commande permet une meilleur lecture des informations en sortie

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

## more informations
curl -XGET 'http://localhost:9200/_cat/nodes?v&h=id,ip,port,v,m'
id   ip         port v     m
nqiB 192.168.0.3 9300 6.0.1 -
ToJE 192.168.0.2 9300 6.0.1 *

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

#### Heap SIZE

```bash

## Affiche la mémoire utilisé par node
curl -sS -XGET "localhost:9200/_cat/nodes?h=heap*&v"

## Deniere version ES
## dans jvm.options

# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms12g
-Xmx12g

# ici on utilise 12Go pour un moteur java lancé

```

#### Assignement de réplicas à 0

Penser à mettre dans les paramètres sur un standalone, le replica à 0
```bash
index.number_of_replicas: 0
```

```bash

for i in $(curl -XGET http://localhost:9200/_cat/shards |grep UNA |awk {'print $1'}) ; do curl -XPUT 'localhost:9200/'$i'/_settings' -d '{"number_of_replicas": 0}' ; done

## ES > 5
curl -XPUT "http://localhost:9200/*/_settings" -H 'Content-Type: application/json' -d '{"settings":{"number_of_replicas":"0"}}'
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

#### Exclude/Include Node ES

```bash
## Exclude
# Cluster 2 nodes : rick & michonne
curl -XPUT 'http://localhost:9200/_cluster/settings?pretty' -d ' { "transient" : { "cluster.routing.allocation.exclude._name" : "rick"  } }'

# Restart du ES pour prise en compte (possibilité de le faire en rolling restart)
/etc/init.d/elasticsearch restart

# permet de voir les exclusions
curl -XGET 'http://localhost:9200/_cluster/settings?pretty'

## Include
curl -XPUT 'http://localhost:9200/_cluster/settings?pretty' -d ' { "transient" : { "cluster.routing.allocation.include._name" : "rick,michonne"  } }'

# Restart du ES pour prise en compte (possibilité de le faire en rolling restart)
/etc/init.d/elasticsearch restart

```


#### Informations Node

https://www.elastic.co/guide/en/elasticsearch/guide/2.x/_monitoring_individual_nodes.html#_indices_section

```bash

curl -XGET "http://localhost:9200/_cluster/allocation/explain?pretty"

curl -XGET "http://localhost:9200/_nodes/stats?pretty=true"

curl -XGET "http://localhost:9200/_nodes/michonne/stats/indices/search?pretty"
{   
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "walkingdead",
  "nodes" : {
    "nqiBMnhwTGWE3GQf-sHiQQ" : {
      "timestamp" : 1523008174882,
      "name" : "michonne",
      "transport_address" : "10.0.16.60:9300",
      "host" : "10.0.16.60",
      "ip" : "10.0.16.60:9300",
      "roles" : [
        "master",
        "data",
        "ingest"
      ],
      "indices" : {
        "search" : {
          "open_contexts" : 0,
          "query_total" : 1550,
          "query_time_in_millis" : 375,
          "query_current" : 0,
          "fetch_total" : 1550,
          "fetch_time_in_millis" : 211,
          "fetch_current" : 0,
          "scroll_total" : 0,
          "scroll_time_in_millis" : 0,
          "scroll_current" : 0,
          "suggest_total" : 0,
          "suggest_time_in_millis" : 0,
          "suggest_current" : 0
        }
      }
    }
  }
}  

# search décrit le nombre de recherches actives (open_context),
# le nombre total de requêtes et le temps passé sur les requêtes depuis le démarrage du nœud.
# Le rapport entre query_time_in_millis / query_total peut être utilisé comme indicateur
# approximatif de l'efficacité de vos requêtes. Plus le rapport est grand,
# plus la durée de chaque requête est longue, et vous devriez envisager un réglage/optimisation.

```
#### Modifications type de la configuration ES

<li> Max thread count & throttle </li>

Si ES prend trop d'io disk sur le système, on peut limiter ses écritures en modifiant le nombre de thread.

On peut également limiter le gouleau.

```bash
# Modifier le max thread count à 1
curl -XPUT 'localhost:9200/_settings' -d '{ "index.merge.scheduler.max_thread_count" : 1}'

# Voir la conf modifié sur l'ensemble des index - remplacer _all par l'index si on veut cibler
curl -XGET 'http://localhost:9200/_all/_settings/index.merge.scheduler.max_*?pretty'

# Modification du gouleau à 20mb
PUT /_cluster/settings
{
    "persistent" : {
        "indices.store.throttle.max_bytes_per_sec" : "20mb"
    }
}

# Pas de limitations
PUT /_cluster/settings
{
    "transient" : {
        "indices.store.throttle.type" : "none" ￼
    }
}

```

#### Header ES

```bash
# Erreur de type : Content-Type header [application/x-www-form-urlencoded] is not supported
# pour des versions supérieurs à 2.0
# -H 'Content-Type: application/json'
curl -XPUT http://localhost:9200/_cluster/settings -H 'Content-Type: application/json' -d '{"transient":{"cluster.routing.allocation.enable":"all"}}'
```

#### Curator

https://www.elastic.co/guide/en/elasticsearch/client/curator/5.x/command-line.html

```bash
curator delete --prefix <prefix> --older-than 7
```
