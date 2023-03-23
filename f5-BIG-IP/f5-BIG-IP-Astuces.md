# Trucs et astuces f5 BIG-IP
## Comment avoir une configuration commune à plusieurs déploiement de f5

Installer et configurer son f5 tel qu'on le souhaite, se connecter en SSH et tapez :

`save sys config file <file name.scf>`

Puis éditer le fichier pour ne conserver que les informations communes aux autres déploiements.

## Voir les 100 VIP qui reçoivent le plus de trafic

bigtop -vname -delay 1 -vips 100 -delta

## iRule pour remplacer le champ host de l'entête HTTP

```
when HTTP_REQUEST {
HTTP::header replace Host "truc.internal"
pool le_pool
}
```

## Activer uniquement TLS1.2 sur la WebGui (et retirer les vieux Cipher)

Source: https://support.f5.com/csp/article/K40232071
```
tmsh
list /sys httpd ssl-ciphersuite
modify /sys httpd ssl-ciphersuite 'ALL:!ADH:!EXPORT:!eNULL:!MD5:!DES:!SSLv2:!SSLv3:!TLSv1:!TLSv1.1'
save /sys config
```

## Passer les VS UDP pour du syslog en Stateless
En Mars 2023, l'équipe de sécurité opérationnelle m'informe qu'ils ne reçoivent plus les logs d'un Cisco ASA.
Après investigation, la "session" UDP est ignorée par le f5. Après une bascule HA, tout est revenu.
L'ASA utilisait comme port UDP source 514 et port destination UDP 514 sur la VIP. Le f5 a décide que la session était trop longue et l'a donc ignoré. 
En passant en Stateless, le VS ne regarde plus les "sessions" UDP. 
