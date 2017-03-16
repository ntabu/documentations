Linux - /dev/shm
==
<br/>

#### Qu'est ce que /dev/shm sous linux

/dev/shm/ est un emplacement mémoire monté de la même manière qu'un disque dur, cependant aucune donnée ne sera enregistrée
sur le disque, tout sera placé directement en mémoire vive. On peux le considérer comme étant l'inverse du SWAP
qui lui est utilisé comme de la RAM mais stocke les données sur le disque.

Il est notamment utilisé comme mémoire partagée car c'est un moyen efficace de transmettre des données entre plusieurs programmes.
Un programme pourra y placer des données qui pourront être utilisées par d'autres processus autorisés.

/dev/shm/ est également très utile pour accélérer la transmission de données sous Linux,
donc accélérer la vitesse de fonctionnement des divers programmes qui l'utilisent.

On l'utilisera par exemple dans des systèmes de cache (pour autant que le cache ne prenne pas toute la RAM disponible).

Il faut cependant l'utiliser avec précaution car il peux causer divers problèmes ou dysfonctionnements.
Comme évoqué ci-dessus il ne faut pas non plus y stocker trop de données sous peine de saturer la mémoire RAM.

#### Sécurisation de /dev/shm/

Comme /tmp, /dev/shm/ peut être une porte ouverte aux rootkits. Il est donc important de le sécuriser.
Le plus simple est de placer un noexec dessus.

```bash

  # Manière temporaire
  mount -o remount,noexec /dev/shm

  # Manière fixe
  vi /etc/fstab
  tmpfs /dev/shm tmpfs defaults,noexec,nosuid,nodev 0
  mount /dev/shm

```
