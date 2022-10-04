# Comment avoir une configuration commune à plusieurs déploiement de f5

Installer et configurer son f5 tel qu'on le souhaite, se connecter en SSH et tapez :

`save sys config file <file name.scf>`

Puis éditer le fichier pour ne conserver que les informations communes aux autres déploiements.

# Voir les 100 VIP qui reçoivent le plus de trafic

bigtop -vname -delay 1 -vips 100 -delta

# iRule pour remplacer le champ host de l'entête HTTP

```
when HTTP_REQUEST {
HTTP::header replace Host "truc.internal"
pool le_pool
}
```
