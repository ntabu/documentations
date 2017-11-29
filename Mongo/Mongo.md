Mongo - TroobleShooting
==
<br/>
Quelques astuces lié à Mongo


#### Afficher l'état d'un Mongo

```bash

    ## connection sur un secondary
    ~ # mongo
      MongoDB shell version: 2.4.10
      connecting to: test
      zombie2:SECONDARY>

      zombie2:SECONDARY> rs.status()

```

#### Forcer un secundary à basculer en primary

```bash

    rs.reconfig(cfg)
    cfg.members = [cfg.members[0]]
    rs.reconfig(cfg, {force : true})
    cfg = rs.conf()

```
