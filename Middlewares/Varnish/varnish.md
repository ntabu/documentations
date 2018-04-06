Middlewares - Varnish
==
<br/>

#### Troobleshooting

```bash

#Show backend Varnish
  varnishadm backend.list

#Check syntax
  varnishd -C -f <file> 	//ex : defaut.vcl

#Test access varnish with curl
  curl -I -H "Host: <domainname>" localhost:<port_listen_varnish>

#boot varnish manually
  DAEMON_OPTS="-a :80 \
             -T :6082 \
             -f /etc/varnish/default.vcl \
                -p thread_pools=6 \
                -p thread_pool_max=4000 \
                -p thread_pool_min=100 \
                -p thread_pool_add_delay=2 \
                -p default_grace=300 \
                -p default_ttl=300 \
                -p ban_lurker_sleep=1 \
                -p sess_workspace=262144 \
             -s malloc,2G"

 varnishd  $DAEMON_OPTS
 pgrep -l varnish

```

#### Astuces

<li> Si le taux d'éviction augmente, cela signifie que le cache évacue les objets plus rapidement et plus vite en raison d'un manque d'espace.

#### Expression regular - req.http.host

https://docs.fastly.com/guides/vcl/vcl-regular-expression-cheat-sheet

#### Ajouter des informations En-tête HTTP pour debug
