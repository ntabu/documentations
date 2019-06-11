Infrastructures - Raid
==
<br/>

Listing des informations lié à la partie RAID

Pour avoir le paquet megacli installer : "apt-get install megaraid-manager"

#### Afficher l etat du RAID

```bash
MegaCli -LDInfo -LALL -aALL -nolog
```
</br>

#### Lister les informations sur la capacité de la batterie

```bash
MegaCli -AdpBbuCmd -GetBbuCapacityInfo -aALL -noLog
```
</br>

#### Lister Disques/Slots/RAID

```bash

MegaCli -PDList -aAll -nolog
#Version courte
MegaCli -PDList -aAll -nolog| egrep "Slot|Device ID|Enclosure"
#Afficher le nombre de slots dispo/utilisés
megacli -EncInfo -aALL -nolog| grep -E 'Slots|Drive'
#Lister les raids
MegaCli -PdList -aALL -nolog | egrep -i "Adapter|Slot|Device ID" | perl -pe 's/^(Enclosure Device ID|Slot Number): (d+)s$/$2t/' | perl -pe 's/DeviceId: //' | perl -pe 's/(^Adapter.*$)/n$1nnEnclIdtSlottIdn---------------------/'
#Lister les disques en fonction du raid
MegaCli -CfgDsply -a0 -nolog
#Lister les device-id
MegaCli -CfgDsply -a0 -nolog| grep "Device Id"
```
</br>

#### Temps de reconstructions du disques

```bash
# Il faut connaitre le numero d'enclosure du raid afin de requêter sur le bon disque : 
# megacli -PDList -aALL -nolog |grep Enclosure 
# Enclosure Device ID: 32
# Puis le numero du slot : [32-1] pour slot1, [32-2] pour slot2 etc ...

megacli -PDRbld -ShowProg -PhysDrv[32:1] -a0 -nolog

```
</br>



#### Test Smart (remplacer N par le device ID)

```bash
smartctl -d megaraid,N -a /dev/sda
```
</br>

#### Gadgets

```bash

# show raid on linux
cat /proc/mdstat

# ??
MegaCli -PdLocate start -physdrv[32:2] -a0

# analyse des disques
omreport storage pdisk controller=0

```

#### Pour HP

```bash
## Raid KO pour HP
hpssacli controller all show config

## cache batterie
hpssacli ctrl slot=0 show | grep Cache

```

