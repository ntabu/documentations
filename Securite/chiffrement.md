Securite - Chiffrement via OpenSSL
==
<br/>

#### Chiffer un fichier

```bash
openssl enc -e -aes-256-cbc -in fichier -out fichier-chiffré
#    enc : on précise qu’on va utiliser un algorithme de chiffrement
#    -e : on chiffre un fichier
#    -aes-256-cbc : l’algorithme, ici AES
#    -in : le nom du fichier à chiffrer
#    -out : le nom de sortie du fichier chiffré

# Un password est demandé (20+ caractères conseillés)
```

#### Déchiffrer un fichier

```bash
openssl enc -d -aes-256-cbc -in fichier-chiffré -out fichier-déchiffré
```
