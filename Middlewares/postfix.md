Middlewares - Postfix
==
<br/>
#### Discard un mail

  1) Add a line to your main.cf
  Code:
̀̀```bash
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
