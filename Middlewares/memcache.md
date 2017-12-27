Middlewares - Memcache
==
<br/>

#### Outils top spécifique à memcache

```bash

wget https://github.com/eculver/memcache-top/archive/master.zip
apt-get install unzip
unzip master.zip

# Puis lancer le launcher

```

On peut également utiliser telnet pour avoir des stats

```bash

telnet 127.0.0.1 11211
 stats

```


#### Troobleshooting

```bash

## Technique de Gitan pour savoir si le memcache est utilisé :D
## On recupere via tcpdump les flux qui match sur 11211 'memcache port'
tcpdump -i any tcp and port 11211 -w /tmp/out.pcap

## On filtre pour connaître les ips qui utilise memcache
tcpdump -nr /tmp/out.pcap | awk '{print$3}' | awk -F "." '{print$1"."$2"."$3"."$4}' | sort -n | uniq -c

```

#### Quelques infos

L’espace "Wasted" disponible correspond à l'espace mémoire attribué pour memcache mais non utilisé.

La mémoire libre (free) est de la mémoire disponible pour toutes les autres applications
