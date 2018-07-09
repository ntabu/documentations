Ansible - Commandes
==
<br/>

#### Commandes utiles

```bash

# pour test les tasks ansible
  --step
# l'option -C permet de check la commande ansible-playbook sans application.
  -C

# forks=1 pour lancer commande en serial
ansible all --sudo -m shell --forks=1 -a 'df -h'

# -m copy permet de copier des fichiers via ansible
ansible -i hosts  all --sudo -m copy -a 'src=roles/haproxy/files/haproxy.cfg dest=/etc/haproxy/'

# ping ansible
ansible localhost -m ping

# installation des rôles galaxy avec les options --force et --ignore-errors
ansible-galaxy install -r requirements.yml --force --ignore-errors

# create/edit le vault (vault est un fichier crypté par ansible
ansible-vault create <file>
ansible-vault edit <file>


```


