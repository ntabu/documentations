Git - Troobleshooting
==
<br/>

#### Impossible d'accepter des MR

```bash

  cd /var/opt/gitlab/git-data

  # backup au cas ou
  rsync -avp gitlab-satellites /opt/backup_satellites/

  # Rm du dépot qu'on n'arrive plus a accepter les MR
  rm -rf <depot>

  # reconstruction des satellites
  gitlab-rake gitlab:satellites:create RAILS_ENV=production

```

