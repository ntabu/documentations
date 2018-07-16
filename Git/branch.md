Git - Branch
==
<br/>

#### Ajout d'une branche

```bash

# Creation de la branche
  git checkout -b <nom_branche>

# Modifications ou ajout de nouveaux fichiers
  git add modules/zabbixagent/files/zabbix/check_spool.sh

# Commit de la modification
  git commit -m "add check file for zabbix"

# Création d'un repo avec trigramme
  git remote add <TRI> git@<domain>:<projet>.git

# Push dans le repo
  git push <tri> <nom_branche>

```
