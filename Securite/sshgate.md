Securite - sshgate
==
<br/>

/opt/sshgate/bin/sshgate-cli -u <user>

#### Gestion utilisateurs

```bash
# Lister les utilisateurs
  user list

# Ajouter utilisateur
  user add <user> mail <email>

# Supprimer utilisateur
  user del

# Afficher la configuration d'un utilisateur
  user <user> display conf

# Lister les machines auquels l'utilisateur a accès
  user <user> list targets

# Modifier la clef ssh publique d'un utilisateur
  user <user> edit sshkey

# Afficher la clef ssh publique de l'utilisateur
  user <user> display sshkey
```

#### Gestion des groupes utilisateurs

```bash

# les commandes sont en partie les mêmes que utilisateurs

# ajouter un utilisateur dans un groupe
  usergroup <group> add user <user>

```


Pour plus d'informations je vous renvoie à ce site qui référence pas mal de commandes sshgate très utile </br>
http://www.starmate.fr/utilisation-de-sshgate/





```
