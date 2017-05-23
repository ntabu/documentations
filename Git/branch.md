Git - Branch
==
<br/>

#### Commande basique

```bash
# Suppression d'un tag
  git tag -d <tag>

# Vérifier la version d'un fichier gitté
  git checkout <nom_fichier>

# Clone d'un projet en ssh
# Attention il faut soit créér <ssh_keygen -b 2048>
# soit ajouter la clé de l'utilisateur qui souhaite cloner le projet
  git clone git@<domain>:<projet>.git

# tagguer une Modifications
  git tag <tag>

# pull (get Merge Request) d'une branche ou projet
  git pull

# push sur une branche (master ou autre)
  git push
  git push --tags

# historique des commits avec graphs
  git log --color --stat --graph --oneline
```

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
