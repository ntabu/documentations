Git - Basic
==
<br/>

#### Commande basique

```bash

# Vérifier la version d'un fichier gitté
  git checkout <nom_fichier>

# Clone d'un projet en ssh
# Attention il faut soit créér <ssh_keygen -b 2048>
# soit ajouter la clé de l'utilisateur qui souhaite cloner le projet
  git clone git@<domain>:<projet>.git

# tagguer une Modifications
  git tag <tag>

# Suppression d'un tag
  git tag -d <tag>

# pull (get Merge Request) d'une branche ou projet
  git pull

# push sur une branche (master ou autre)
  git push
  git push --tags

# annulation du git push
  git reset --hard HEAD~1

# changer de branches
  git checkout <nom_branche>

# voir le dernier commit
  git branch -v

# historique des commits avec graphs
  git log --color --stat --graph --oneline

```

#### Global

```bash

# Forcer la communication vers le git en tls1.2

  git config --global --list
  git config --global --unset http.sslVersion
  git config --global --add http.sslVersion tlsv1.2

```


