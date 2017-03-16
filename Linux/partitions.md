Linux - Partitions
==
<br/>

Procèdure pour ajouter un disque

<br/>

#### Sous debian

```bash
root@liberty:/# df -h
Sys. de fichiers               Taille Utilisé Dispo Uti% Monté sur
/dev/vda5                        2,8G    229M  2,4G   9% /
udev                              10M       0   10M   0% /dev
tmpfs                            1,6G     17M  1,6G   2% /run
/dev/dm-1                        3,7G    651M  2,9G  19% /usr
tmpfs                            4,0G       0  4,0G   0% /dev/shm
tmpfs                            5,0M       0  5,0M   0% /run/lock
tmpfs                            4,0G       0  4,0G   0% /sys/fs/cgroup
/dev/vda1                        464M     33M  403M   8% /boot
/dev/mapper/systemvm-var         4,5G    239M  4,0G   6% /var
/dev/mapper/data-var_lib_mysql    89G    8,0G   76G  10% /var/lib/mysql
/dev/mapper/systemvm-home        1,9G    3,0M  1,8G   1% /home
/dev/mapper/systemvm-var_log     4,6G     25M  4,3G   1% /var/log
/dev/mapper/systemvm-tmp         1,9G    2,9M  1,8G   1% /tmp

root@liberty:/# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  data       1   1   0 wz--n- 100,00g 10,00g
  systemvm   1   6   0 wz--n-  46,66g 27,70g
```

</br>

On peut constater sur ce serveur qu'il y a deux VGS (volume groups). </br>
Imaginons que vous voulons augmenter le /var/lib/mysql. </br>
===> /dev/mapper/<strong>data</strong>-var_lib_mysql, on voit qu'il fait partie du VG data. </br>

```bash

# On ajoute 5go à /var/lib/mysql
lvresize -L+5g /dev/mapper/data-var_lib_mysql && resize2fs /dev/mapper/data-var_lib_mysql
```

Si on veut ajouter 5go au /var/log </br>

```bash

# On ajoute 5go à /var/log
lvresize -L+5g /dev/mapper/systemvm-var_log && resize2fs /dev/mapper/systemvm-var_log
```

</br>

Pour ajouter un disque sur Proxmox (sous interface web) <br/>
On séléctionne la VM où l'on souhaite ajouter un nouveau disque dans Server view <br>
Dans l'onglet Hardware/Add/Hard Disk </br>
On séléctionne le Storage, le format (qcow,qcow2,...), et la taille DD etc... Puis Add </br>
On devrait retrouver dans la description quelques choses comme ça : </br>

```bash

virtio1: VMs_hypliberty:115/vm-115-disk-0.qcow2,format=qcow2,size=50Gsize=50G
```

</br>

Sur la VM :

```bash
# On récupère l'info via un dmesg -T
[19783646.723135]  vdc: unknown partition table

# Création du physical volume
pvcreate /dev/vdc

# Extension du VG que l'on souhaite agrandir
vgextend systemvm /dev/vdc
```
