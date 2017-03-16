APT
==
<br/>

Sans demarrage des services lors d'un apt-get install

<br/>

```bash
vi /usr/sbin/policy-rc.d <br/>
    #!/bin/sh  
    exit 101
chmod +x /usr/sbin/policy-rc.d
```

#### Attention

ne pas oublier de supprimer le fichier une fois le paquet installé.
