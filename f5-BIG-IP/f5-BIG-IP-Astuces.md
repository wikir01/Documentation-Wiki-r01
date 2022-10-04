Comment avoir une configuration commune à plusieurs déploiement de f5:

Installer et configurer son f5 tel qu'on le souhaite, se connecter en SSH et tapez :

`save sys config file <file name.scf>`

Puis éditer le fichier pour ne conserver que les informations communes aux autres déploiements.

bigtop -vname -delay 1 -vips 100 -delta

# iRule pour remplacer le champ host de l'entête HTTP

`when HTTP_REQUEST {

HTTP::header replace Host "irs-api-gateway-q-pfr5-irs-agw-comp.apps.q-k8s-pfr5-irs.z2.r02.local"

pool apigw.irs-qual.PFR5-COMP_pool
}`
