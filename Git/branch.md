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

#### Importer répertoire existant dans le GIT

```bash 

# dans le répetoire existant 
# initialisaiton du git
  git init

# ajout des fichiers et répertoire présent + commit
  git add *
  git commit -m "add new repositeries"

# configuration du git pour l'ajout dans le bon repositery
  git config --global user.email "<mail>"
  git config --global user.name "Little Pug"
  git push --set-upstream https://github.com/ntabu/indus.git master


```

#### Clone GIT à plusieurs branches

2 solutions : 


```bash 

# script bash 

    for branch in $(git branch --all | grep '^\s*remotes' | egrep --invert-match '(:?HEAD|master)$'); do
      git branch --track "${branch##*/}" "$branch"
    done


# a la mano

    git branch -a
      *  master
         remotes/origin/HEAD -> origin/master
         remotes/origin/master
         remotes/origin/branch1
         remotes/origin/branch2
         remotes/origin/branch3

    git checkout origin/branch2
    git checkout branch2
    ——> git branch
        master
        * branch2

```
