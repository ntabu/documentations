Infrastructures - Raid
==
<br/>

Listing des informations lié à la partie RAID

#### Afficher l etat du RAID

```bash
MegaCli -LDInfo -LALL -aALL
```
</br>

#### Lister les informations sur la capacité de la batterie

```bash
MegaCli -AdpBbuCmd -GetBbuCapacityInfo -aALL -noLog
```
</br>

#### Lister Disques/Slots/RAID

```bash

MegaCli -PDList -aAll
#Version courte
MegaCli -PDList -aAll | egrep "Slot|Device ID|Enclosure"
#Afficher le nombre de slots dispo/utilisés
megacli -EncInfo -aALL | grep -E 'Slots|Drive'
#Lister les raids
MegaCli -PdList -aALL | egrep -i "Adapter|Slot|Device ID" | perl -pe 's/^(Enclosure Device ID|Slot Number): (d+)s$/$2t/' | perl -pe 's/DeviceId: //' | perl -pe 's/(^Adapter.*$)/n$1nnEnclIdtSlottIdn---------------------/'
#Lister les disques en fonction du raid
MegaCli -CfgDsply -a0
#Lister les device-id
MegaCli -CfgDsply -a0 | grep "Device Id"
```
</br>

#### Test Smart (remplacer N par le device ID)

```bash
smartctl -d megaraid,N -a /dev/sda
```
</br>

#### Gadgets

```bash
MegaCli -PdLocate start -physdrv[32:2] -a0
```
