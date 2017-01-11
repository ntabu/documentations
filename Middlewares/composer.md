Middlewares - Composer
==
<br/>
```bash
# Download l'installer dans le répertoire courant
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Vérifie l'installer
php -r "if (hash_file('SHA384', 'composer-setup.php') === '55d6ead61b29c7bdee5cccfb50076874187bd9f21f65d8991d46ec5cc90518f447387fb9f76ebae1fbbacf329e583e30') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# Run de l'installer avec la spécification de la version (on peut spécifier aussi le directory)
php composer-setup.php --version=1.2.1

# Remove de l'installer
php -r "unlink('composer-setup.php');"

```

Puis on place le .phar dans le path
```bash
mv composer.phar /usr/local/bin/composer
composer --version
Composer version 1.2.1 2016-09-12 11:27:19
```
