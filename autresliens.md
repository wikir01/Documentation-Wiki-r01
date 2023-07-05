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
