Middlewares - Varnish - Entêtes
==
<br/>

#### Ajouter des informations dans l’en-tête HTTP pour debugger

Le but est de pouvoir vérifier si l'objet vient du cache ou pas.

Dans le vcl_deliver :

```bash

if (obj.hits > 0){
      set resp.http.X-Varnish-Cache = "HIT";
}else{
      set resp.http.X-Varnish-Cache = "MISS";
}

# Pour vérifier si la request est passée par la routine PASS

sub vcl_pass {
        set req.http.X-marker = "pass" ;
}

sub vcl_fetch {
        if (req.http.X-marker == "pass" ) {
                unset req.http.X-marker;
                set beresp.http.X-marker = "pass" ;
        }
}

sub vcl_deliver {
        if (resp.http.X-marker == "pass" ) {
                remove resp.http.X-marker;
                set resp.http.X-Varnish-Cache = "PASS";
        }
}

# Pour voir le TTL dans les Headers des objets :

sub vcl_fetch {

set beresp.http.X-Varnish-TTL = beresp.ttl;

}

# Pour voir le nom du serveur où se trouve varnish

sub vcl_fetch {

set beresp.http.X-Served-By = server.hostname;

}

# Pour voir le backend dans les en-têtes des objet

sub vcl_fetch {

set beresp.http.Served-By = beresp.backend.name ;

}

# Pour voir le host et l’url dans les en-têtes des objets
sub vcl_fetch {
set beresp.http.X-host = req.http.host;
set beresp.http.X-url = req.url;
}

```
