Middlewares - Varnish - Entêtes
==
<br/>

#### Ajouter des informations dans l’en-tête HTTP pour debugger

<li> Vérifier si l'objet vient du cache ou pas. </li>

```bash
sub vcl_deliver {

  if (obj.hits > 0){
        set resp.http.X-Varnish-Cache = "HIT";
  } else{
        set resp.http.X-Varnish-Cache = "MISS";
  }
}

```


<li> vérifier si la request est passée par la routine PASS </li>

```bash
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

```

<li> voir le TTL dans les Headers des objets </li>

```bash

sub vcl_fetch {

  set beresp.http.X-Varnish-TTL = beresp.ttl;

}

```

<li> le nom du serveur où se trouve varnish </li>

```bash

sub vcl_fetch {

set beresp.http.X-Served-By = server.hostname;

}

```

<li> le backend dans les en-têtes des objet </li>

```bash

sub vcl_fetch {

set beresp.http.Served-By = beresp.backend.name ;

}

```

<li> voir le host et l’url dans les en-têtes des objets </li>

```bash

sub vcl_fetch {
set beresp.http.X-host = req.http.host;
set beresp.http.X-url = req.url;
}

```
