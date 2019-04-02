Middlewares - Haproxy - Stats
==
<br/>

#### Voir les stats du haproxy

```bash

# englober dans du watch pour avoir les infos en boucle.
# affiche les requêtes pour l'ensemble des backends.
echo "show stat"|nc -U /run/haproxy/admin.sock | cut -d "," -f 1,2,5-11,18,24,27,30,36,50,37,56,57,62 | column -s, -t

# englober dans du watch pour avoir les infos en boucle.
# informations sur le haproxy en local
echo "show info" | nc -U /run/haproxy/admin.sock

```
