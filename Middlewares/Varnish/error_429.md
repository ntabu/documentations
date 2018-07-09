Middlewares - Varnish - Erreur 429
==
<br/>


#### Suppression des cookies pour WorldPress

```bash
if (req.http.cookie) {
	if (req.http.cookie ~ "(wp-login-|wordpress_|wp-settings-)") {
        	return(pass);
        } else {
                unset req.http.cookie;
        }
}

```

