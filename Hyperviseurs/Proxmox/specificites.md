Proxmox - Specifiques
==
<br/>

Des commandes utiles

#### Basiques  

```bash
# Afficher listes des VMs
qm li

# Stopper VM
qm stop <vmid>

# Start VM
qm start <vmid>

# Delock VM
qm unlock <vmid>


```

</br>

#### Ajouter un disque via shell

```bash
modprobe acpiphp pci_hotplug  # sur la VM

## Ajout qcow via qm monitor

qemu-img create -f qcow2 -o preallocation=metadata vm-100-disk-2.qcow2 50G

# Proxmox ≤ 3.2
qm monitor <id_vm>
pci_add auto storage file=/VMs/images/100/vm-100-disk-2.qcow2,if=virtio

# Proxmox ≥ 3.3
drive_add 0 file=/VMs/images/100/vm-100-disk-2.qcow2,format=qcow2,id=drive-virtio-disk2,if=none
device_add virtio-blk-pci,scsi=on,drive=drive-virtio-disk2

# ne pas oublier d'ajouter dans le conf en dur :
vi /etc/pve/qemu-server/<VMID>.conf
virtio1: VMs_h2op009:100/vm-100-disk-1.qcow2,format=qcow2

```
</br>

#### Scan des qcow + repartions des corruptions de données

```bash

# info du disque
qemu-img info vm-102-disk-1.qcow2

# scan du disque
qemu-img check /VMs/images/117/vm-117-disk-0.qcow2

# repare tous les block corrupted
qemu-img check -r all /VMs/images/117/vm-117-disk-0.qcow2

```

</br>

#### Convertions disques  

```bash
qemu-img convert -p -f qcow2 <source> -O qcow2 <dest>
```

</br>

#### PVESH   

```bash
# voir les hyperviseurs dans le cluster
pvesh get /nodes

# voir le cluster Name et plus d'infos sur le cluster
pvecm status

# test acces API proxmox
curl https://<ip>:8006/api2/json/access/ticket -k -d 'username=root@pam&password=<password>'

# infos VM
pvesh get nodes/<nom_hyperviseur>/qemu/114/status/current

# ajout d'un disque via pvesh
pvesh create /nodes/<nom_hyperviseur>/qemu/604/config -virtio2 'VMs_HYP6:50,format=qcow2'

```

</br>
