### Outils

Joplin pour prendre des notes

Telegraph + InfluxDB + Grafana pour avoir de belles courbes de débit sur les switches.

### Liens

https://github.com/BaptisteBdn/docker-selfhosted-apps/tree/main/nextcloud

https://www.reddit.com/r/selfhosted/comments/s7pmg2/advice\_on\_cheap\_domain\_name/

https://www.reddit.com/r/selfhosted/comments/s6x9e7/cloudflare\_tunnels\_alternative\_to\_vpn\_or\_port/

https://tld-list.com/

https://gen.xyz/register

https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide

https://crowdsec.net/

https://jellyfin.org/

Pour télécharger un fichier reconnu comme étant un virus faux positif: https://www.eicar.org/download-anti-malware-testfile/

### **RP:**

https://traefik.io/

https://www.envoyproxy.io/try/

## Autre

powertop

tlp

sur top maj+1 pour afficher tous les CPU

## Serveur FTP ouvert pour faire des tests de DPL

https://dlptest.com/ftp-test/

Dans vim pour supprimer 100 lignes: d100d

## Netcat 

echo ${HOSTNAME} | ncat -vz @ServeurDest port

## openssl voir certificats

openssl s_client –connect siteweb.com:443 -showcerts

## Voir le contenu d'un certificat avec certutil

certutil -v

## Chercher sur Redhat les différentes versions d'un package

yum --showduplicates list lftp

## Trouver les groupes d'un utilisateur dans l'AD (sans avoir accès à l'AD)

NET USER nomutilisateur /DOMAIN
ou gpresult /r

## Pour debug du Windows

Procmon pour voir tout ce qu'il se passe terme de processus / applications.

https://learn.microsoft.com/fr-fr/sysinternals/downloads/procmon

## Utiliser des mots de passes chiffrés dans un script bash

Procédure pour utiliser un mot de passe chiffré en bash

Note: c'est pas super propre mais c'est toujours mieux que d'avoir le mot de passe en clair dans un .sh

Voici les différentes étapes:


### 1- Création d'un répertoire secrets où on appliquera des droits limités (400 lecture unique de l'utilisateur qui fait tourner le script)

```
mkdir secrets
```

### 2- Création d'un fichier hash.key qui va contenir la clé de chiffrement qui sera hash en base64

```
echo 'CléSuperSecrète' | base64 > secrets/hash.key
```

### 3- Création du fichier encrypted.key qui contiendra le secret (le mot de passe du compte de service svc-p-swt-0001) et qu'on va chiffrer

```
echo 'mot_de_passe_compte_de_backup' > secrets/encrypted.key
```

### 4- Mise en place des droits lecture seul sur le fichier hash.key

```
chmod 400 secrets/hash.key
```

### 5- Chiffrement du mot de passe du compte de service dans encrypted.enc

Pour RHEL 7:
```
var_key=$(cat secrets/hash.key | base64 --decode)
openssl enc -e -aes-256-cbc -in secrets/encrypted.key -out secrets/encrypted.enc -pass pass:$var_key
```

Pour RHEL 8:
```
var_key=$(cat secrets/hash.key | base64 --decode)
openssl aes-256-cbc -e -salt -pbkdf2 -iter 10000 -in secrets/encrypted.key -out secrets/encrypted.enc -k $var_key
```

###6- Suppression du fichier encrypted.key qui n'est pas chiffré

```
rm secrets/encrypted.key
```

### 7- Mise en place des droits lecture seul du fichier encrypted.enc

```
chmod 400 secrets/encrypted.enc
```

### 8- Utilisation du mot de passe chiffré dans le script de backup

A partir de ce moment, il est possible d'utiliser le fichier encrypted.enc qui contient le mot de passe du compte de backup dans les scripts de cette façon:

Pour RHEL 7:
```
var_key=$(cat secrets/hash.key | base64 --decode)
secret=$(openssl enc -d -aes-256-cbc -in secrets/encrypted.enc -pass pass:$var_key)
```

Pour RHEL 8:
```
var_key=$(cat secrets/hash.key | base64 --decode)
secret=$(openssl aes-256-cbc -d -salt -pbkdf2 -iter 10000 -in secrets/encrypted.enc -k $var_key)
```

La variable $secret contient le mot de passe du compte de service et peut être utilisée dans le script.
