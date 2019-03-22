Securite - Iptables
==
<br/>

#### Activation des logs iptables

```bash

# Log des drops
# Log iptables
#iptables -N LOGGING
#iptables -A INPUT -j LOGGING
#/sbin/iptables -A INPUT -m limit --limit 60/min -j LOG --log-prefix "[DROP]" --log-level 4


```
