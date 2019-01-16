Réseaux - Keepalived
==
<br/>

Mise en place d'un load balancing en haute dispo

#### Configuration node1 et node2

```bash
## node 1 avec un priority à 100
vrrp_instance VIP_1 {
        state MASTER
        interface eth0
        priority 100
        advert_int 1
        lvs_sync_daemon_interface eth0
        virtual_router_id 66
        virtual_ipaddress {
                10.0.16.200 brd 10.0.16.255 dev eth0 scope global
        }
}

virtual_server 10.0.16.200 80 {
    delay_loop 5
    lb_algo rr
    lb_kind NAT
    protocol TCP

    real_server 10.0.16.12 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
    real_server 10.0.16.13 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
}

virtual_server 10.0.16.200 443 {
    delay_loop 5
    lb_algo rr
    lb_kind NAT
    protocol TCP

    real_server 10.0.16.12 443 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
    real_server 10.0.186.13 443 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
}

```

```bash
## node 2 avec un priority à 50
vrrp_instance VIP_1 {
        state BACKUP
        interface eth0
        priority 50
        advert_int 1
        lvs_sync_daemon_interface eth0
        virtual_router_id 66
        virtual_ipaddress {
                10.0.16.200 brd 10.0.16.255 dev eth0 scope global
        }
}

virtual_server 10.0.16.200 80 {
    delay_loop 5
    lb_algo rr
    lb_kind NAT
    protocol TCP

    real_server 10.0.16.12 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
    real_server 10.0.16.13 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
}

virtual_server 10.0.16.200 443 {
    delay_loop 5
    lb_algo rr
    lb_kind NAT
    protocol TCP

    real_server 10.0.16.12 443 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
    real_server 10.0.186.13 443 {
        weight 1
        TCP_CHECK {
            connect_timeout 1
        }
    }
}

```

#### Test

```bash

## vérifier le bon fonctionnement 

root:~ # ipvsadm -Ln -t 10.0.186.200:80
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.16.200:80 rr
  -> 10.0.16.12:80               Masq    1      3          1
  -> 10.0.16.13:80               Masq    1      4          1

## si le node1 tombe, le node2 devient le master
## si le keepalived du node1 est stoppé, le node2 devient le master
## avec le priority, si node2 est redemarré il ne bascule pas
## si le node2 est master et qu'on demarre le node1, le node1 devient master

```
