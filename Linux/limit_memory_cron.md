Linux - limit memory
==
<br/>
#### Comment limiter tous les crons au niveau memoire
<li> créer un utilisateur cron (crontest)
<li> ajouter la ligne suivante dans /etc/security/limits.conf
```bash
# Pour 50mo
crontest        hard    as              50000
```
<li> Faire un petit script qui alloue de la mémoire (en C ou autres langages)
```bash
#include <malloc.h>
#include <unistd.h>
#include <memory.h>
#define MB 1024 * 1024
int main() {
        int size;
        size = 0;
    while (1) {
        void *p = malloc( 10*MB );
        memset(p,0, 10*MB );
        size = 10+size;
        printf("Using %d MB\n",size);
        sleep(1);
    }
}

gcc mem.c -o mem
cp mem /bin/mem
```

<li> puis tester
```bash
* * * * * crontest /bin/mem
#voir dans dmesg le comportement (kill out of memory)
# ou segfault dans /var/log/syslog
```
<br/>
#### Avec Cgroups
