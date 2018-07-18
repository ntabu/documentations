ElasticSearch - Généralités
==
<br/>
ES est un moteur de recherche composé de plusieurs éléments.</br>
Un moteur de recherche sur les indexes, un moteur d indexation de documents. </br>
Un cluster est composé de node (instance d ES), un index est une collection de documents qui ont des caractéristiques similaires.</br> Les shards sont des partitions où sont stockés les documents constituant un index. </br>
Les réplica sont des copies additionnels d un index dans un cluster ES. </br>
Par défaut, chaque index est répartie sur 5 shards primaires et 1 réplica. </br>
La com passe par le port 9300. </br>
Il faut que java soit installé dessus pour fonctionner. </br> </br>
Petit Tuto interessant.
http://zenika.developpez.com/tutoriels/java/installation-configuration-elasticsearch/

#### Arborescence ES.
```bash
//Tree (/etc/elasticsearch)
	bin
	config	(elasticsearch.yml et logging.yml)
	lib
		//Apres premier lancement
			data		(données indexées)
			logs
			work		(contient des fichiers temporaires au fct moteur de recherche)
```
#### Plugin ES Web
```bash
./bin/plugin -install mobz/elasticsearch-head	//interface admin Elastic
```
#### Compréhension
```bash
# curl -XGET http://localhost:9200/_cat/shards 
sf_zombi_2016-04-12-200732 4 r STARTED 251532   2gb 192.168.0.2 srvweb01 
sf_zombi_2016-04-12-200732 4 p STARTED 251532   2gb 192.168.0.3 srvweb02 
```
Descriptions (par colonne):

<ul>
	<li>nom index</li>
	<li>numero shard</li>
	<li>type (primaire ou replica)</li>
	<li>statut</li>
	<li>ID</li>
	<li>taille</li>
	<li>adresse du noed</li>
	<li>nom du noeud</li>
</ul>

Exemple de configurations : (/etc/elasticsearch/elasticsearch.yml)
<br/>
```bash
cluster.name:			//nom du cluster
node.name:			//nom du noeud ES
path.conf:			//chemin du fichier de conf ES
path.logs:			//chemin des logs ES
http.enabled 			//(active/désacte le support HTTP REST) 
node.data 			//(active/désactive le stockage de données indexées)
```
#### Paramètres ES 
On peut définir la mémoire allouée à ES dans /etc/default/elasticsearch
```bash
# Heap Size (defaults to 256m min, 1g max)
ES_HEAP_SIZE=4g
```
**Attention**

ES s'attribue la totalité de la mémoire allouée.

Ex:

>Si on a un machine de 8go, et qu'on a attribué à ES 4go<br/>
>Mettons que php grimpe et s'attribue 6go de traitements.<br/>
>ES tombe car il lui faut ses 4go.<br/>
>Message d'erreur : 
```bash
dmesg -T
Out of Memory: java ...
```
