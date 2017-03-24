Middlewares - Postfix
==
<br/>

#### Discard un mail

  1) Add a line to your main.cf
  Code:

```bash
    header_checks = regexp:/etc/postfix/header_checks
```

  2) Create a new file, /etc/postfix/header_checks
  Code:

```bash
/addresstoblock@yourdomain\.com/    DISCARD
```

  3) Restart postfix
  Code:

```bash
/etc/init.d/postfix restart
```

#### Limit nb mail send by IP

On peut paramétrer une limite du nb de mails envoyés sur une intervalle de temps. </br>

Ex : limite de 10 mails/min

```bash
vi /etc/postfix/main.cf


# Whitelist
smtpd_client_event_limit_exceptions = 127.0.0.0/8
# Max 10 mails/IP/anvil_rate_time_unit
smtpd_client_message_rate_limit = 10
# Reject pipelining (speed-up mail sending)
smtpd_data_restrictions = reject_unauth_pipelining
smtpd_delay_reject = yes
# Reset counter each minute
anvil_rate_time_unit = 60s
# Log statistics each 5 minutes
anvil_status_update_time = 600s


# Puis reload de la conf
postfix reload

```

Chaque fois qu'un client atteint cette limite, il reçoit une erreur spécifique et le mail ira en deferred </br>

```bash
Error: too much mail from <ip>

Sep  2 16:04:07 machine.local postfix/smtpd[25359]: warning: Message delivery request rate limit exceeded: 55 from domain1.com[<ip>] for service smtp
```

Listing des machines qui ont atteint la limite sur la journée en cours </br>

```bash

/bin/cat /var/log/mail.log | grep "Message delivery request rate limit exceeded" |  grep "`date +%b' '%e`" | while read ligne ; do echo $ligne | awk '{print $(NF-5),"\t",$(NF-3)}' ; done | sort -k1 -g | uniq > /tmp/liste_rate_limit && for i in `/bin/cat /tmp/liste_rate_limit | cut -d'[' -f2 | cut -d ']' -f1 | sort | uniq`; do /bin/cat /tmp/liste_rate_limit | grep "$i" | sort -k1 -g | tail -1 ; done | sort -k1 -gr

```

#### Changer l'adresse ou le domaine de destination postfix

```bash

# passe par trivial-rewrite
# /etc/postfix/main.cf
smtp_generic_maps = regexp:/etc/postfix/generic

# /etc/postfix/main.cf
/^.*@domain1.com/ user1@domain1.com # renvoyer le domaine vers un utilisateur spécifique
/^.*@domain1.com/ $1@domain2.com    # renvoyer les mails de l'utilisateur de domain1.com vers domain2.com
/^.*@.*/ user1@domain1.com          # tout renvoyer vers user1@domain1.com

# activation de la conf
postmap /etc/postfix/generic
/etc/init.d/postfix restart

```

#### Empecher les mails d'être délivrés

Afin d’empêcher les mails de certains domaines d’être délivrés, par exemple en cas de maintenance ou de migration du domaine, on peut laisser ces mails dans la queue en les mettant en status « HOLD » dans postfix. </br>

Pour passer un domaine en HOLD, il faut modifier le fichier /etc/postfix/main.cf tel que ci-dessous et redémarrer postfix : </br>

```bash
smtpd_recipient_restrictions =
    ...
    check_recipient_access hash:/etc/postfix/hold

# /etc/postfix/hold
example1.com        HOLD
example2.com         HOLD

# application des changements
postmap /etc/postfix/hold

# pour libérer les mails en HOLD
postsuper -r ALL

```
