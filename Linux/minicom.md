Linux - Minicom
==
<br/>

Mettre en place minicom avec un script ou sans.

Pour le script : https://github.com/ntabu/script/blob/master/connect-vm

<br/>

VMs :

```bash
# pour faire fonctionner le script c'est une version 2.7 mini
# sur l'hyperviseur  
apt-get install minicom

# Dans la VM :
cat /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0,19200n8"
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"

# puis on upgrade le grub. Nécessite un reboot de la VM
update-grub

```

Hyperviseurs :

```bash

# Fichier de conf de la VM proxmox
# /etc/pve/qemu-server/<vmid>.conf
# en debut de fichier
args: -serial unix:/var/run/qemu-server/<vmid>.serial,server,nowait

# restart VM
qm stop <vmid> ; qm start <vmid>

# en mode manuelle
minicom -D unix#/var/run/qemu-server/100.serial

# avec le script préalablement disposé dans /usr/local/bin
connect_VM --connect <vmid>

```
