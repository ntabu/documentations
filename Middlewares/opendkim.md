Middlewares - OpenDKIM
==
<br/>

#### Installation et prérequis postfix

```bash
# Installation des paquets
apt-get install opendkim

# déclaration dans /etc/postfix/main.cf
milter_default_action = accept
milter_protocol = 2
smtpd_milters = inet:127.0.0.1:8895
non_smtpd_milters = inet:127.0.0.1:8895

```

Parametrages d'opendkim.conf </br>

```bash

# dans /etc/opendkim.conf
Syslog                  yes
SyslogSuccess           Yes
UMask                   002

ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts
InternalHosts           refile:/etc/opendkim/TrustedHosts

KeyTable                refile:/etc/opendkim/KeyTable
SigningTable            refile:/etc/opendkim/SigningTable

Domain                  domain1.zombie.com,domain2.zombie.com
Selector                default
Canonicalization        simple
Mode                    sv
SignatureAlgorithm      rsa-sha256
SubDomains              no
Background              yes
DNSTimeout              5

PidFile                 /var/run/opendkim/opendkim.pid
UserID                  opendkim:opendkim

OversignHeaders         From

```

Apres le parametrage, ne pas oublier d'attribuer les droits opendkim: sur les fichiers de conf </br>

Dossier opendkim
```bash

# creation du Dossier
mkdir opendkim

# creation des fichiers et dossiers suivants dans /etc/opendkim
drwxr-xr-x 4 root     root     4,0K févr.  1 16:48 keys
-rw-r--r-- 1 opendkim opendkim  217 févr.  2 13:43 KeyTable
-rw-r--r-- 1 opendkim opendkim   99 févr.  2 13:49 SigningTable
-rw-r--r-- 1 opendkim opendkim   37 févr.  1 16:41 TrustedHosts

# dans keys
# default.txt et default.private sont les clés privés et public du domain
6499 4 drwxr-xr-x 4 root     root     4096 févr.  1 16:48 .
6288 4 drwxr-xr-x 3 root     root     4096 févr.  2 14:47 ..
6453 4 drwxr-xr-x 2 opendkim opendkim 4096 févr.  1 16:46 domain1.zombie.com
6621 4 drwxr-xr-x 2 opendkim opendkim 4096 févr.  2 11:21 domain2.zombie.com

keys/domain1.zombie.com:
total 16
6453 4 drwxr-xr-x 2 opendkim opendkim 4096 févr.  1 16:46 .
6499 4 drwxr-xr-x 4 root     root     4096 févr.  1 16:48 ..
6619 4 -r--r----- 1 opendkim opendkim  887 févr.  1 16:46 default.private
6620 4 -rw-r--r-- 1 opendkim opendkim  311 févr.  1 16:46 default.txt

keys/domain2.zombie:
total 16
6621 4 drwxr-xr-x 2 opendkim opendkim 4096 févr.  2 11:21 .
6499 4 drwxr-xr-x 4 root     root     4096 févr.  1 16:48 ..
6622 4 -r--r----- 1 opendkim opendkim  887 févr.  1 16:46 default.private
6623 4 -rw-r--r-- 1 opendkim opendkim  306 févr.  1 16:46 default.txt


```

Parametrages KeyTable </br>

```bash
default._domainkey.domain1.zombie.com zombie.com:default:/etc/opendkim/keys/domain1.zombie.com/default.private

default._domainkey.domain2.zombie.com domain2.zombie.com:default:/etc/opendkim/keys/domain2.zombie.com/default.private

```
Parametrages SigningTable </br>

```bash
*@domain1.zombie.com default._domainkey.zombie.com
*@domain2.zombie.com default._domainkey.zombie.com


```

Parametrages TrustedHosts </br>

```bash

127.0.0.1
::1
localhost
<ip_network_listen>


```
