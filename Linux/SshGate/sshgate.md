SSHGATE - Bastion
==
<br/>

Equivalent de sshportal

Documentations : http://www.starmate.fr/utilisation-de-sshgate/

#### Pratiques

```bash

## acces en scp via sshgate pour un directory - retirer -r pour un fichier
# il faut partir du principe que l’utilisateur se connecte dans son répertoire défini dans son /etc/passwd
# pour le coup <login> c’est dans le /home/<login>
# il faut donc utiliser les ../.. pour se diriger vers le /var/www/toto
scp -r -P <port> sshgate@<ip_sshgate>:<login>@<host>/../../var/www/toto/* .

```

#### Utilisation

```bash

# ajout d'un user sur le sshgate
  user add <nom> mail <mail>
  usergroup <groupe> add user <nom>
  usergroup <groupe> list users

# edit de la clé sshkey (attention supprime la clé actuelle si edit de la clé
  user <user> edit sshkey

# ajout d'une target 
# jonsnow : host ; nightswatch : usergroup/login
  sshGate >  target add jonsnow
  Use the sshGate default sshkey for this target host [Y] ?   Y
  NOTICE: Public ssh key of 'jonsnow' can't be installed on 'nightswatch@jonsnow'. Install it manually

  sshGate >  target jonsnow ssh add login nightswatch
  sshGate >  target nightswatch@jonsnow access add usergroup nightswatch

```

#### Troobleshooting

```bash

# delock du bastion
 find . -name "*.lock"
 /opt/sshgate # lsof ./logs/current_session.log.mutex.lock
# suppression ou move du lock
 mv ./logs/current_session.log.mutex.lock ./logs/current_session.log.mutex.lock.old

# automisation de la suppresion des lock 
 vi /etc/cron.d/rm_lock_bastion
 0/20 * * * * root for i in $(find /opt/sshgate/ -name "*lock*" |egrep -v "lock.testcase|lock.lib.sh") ; do rm $i ; done

```
