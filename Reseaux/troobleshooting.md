Réseaux - Troobleshooting
==
<br/>
Quelques commandes intéressantes pour dépanner un réseau.
#### Rechercher d'informations IP
```bash
# dig - Domain Information Groper
# Interroge le serveur DNS de son choix (par défaut celui du /etc/resolv.conf)
  dig <hostname>
# Reverse llookups
  dig -x <ip>
# Autres outils
  host
  nsloockup
# whois - service de recherche fourni par les registres internet
# donne une multitude d'information sur une ip demandée.
  whois <ip>
```

#### TCPDUMP
```bash

# Je recupère une trame de n'importe quel interface sur l'host <ip>
# ou le sous-réseau 10.... et sur le port 80
# -w permet de redirigé dans un fichier.
# s0 permet de spécifier la capture du paquet au complet
  tcpdump -vi any tcp and '(host 31.15.26.212 or net 10.0.174)' and port 80 -s0  -w /tmp/out.pcap
# Lecture du tcpdump capturé
  tcpdump -nr  /tmp/out.pcap | less

  tcpdump -vvvv host localhost -i eth0 -p 80

  tcpdump src 192.168.1.100 and dst 192.168.1.2 and port ftp

# Avec un timeout de 1000s
  timeout 1000 tcpdump -i bond0 host <ip> and port 80 -s0
```
