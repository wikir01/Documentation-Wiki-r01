Créer un VS (guide complet pour un VS L7 HTTPS)
-----------------------------------------------

### Générer CSR

Hélas, le fortiADC n'a pas le champ SAN dans son super formulaire. Il est nécessaire de le faire à la main avec OpenSSL si vous avez besoin d'un champ SAN. Une fois que la clé, et la CSR sont générées, il faut signer cette CSR avec la PKI entreprise.

Une fois que le certificat est signé, alors suivez le reste de la procédure:

Allez dans votre VDOM (ici technique), aller dans System, Manage Certificates sous Certificate. Enfin, dans le menu en haut de la fenêtre, aller dans l'onglet Local Certificate.

Cliquer sur Import.

\[Capture Manage Certificates\]

Choisissez le type d'import (ici Certificat),

Donnez un nom suivant le format AAAAMMJJ\_nomducertificat

Dans Certificate File, mettez le certificat généré par la Sécop

Dans l'onglet Key File, mettez la clé privée (normalement c'est interdit de déplacer des clés privées mais l'outil ne permet de faire autrement)

Cliquez sur Save.

\[Capture Local Certificate\]

### Mettre le certificat dans un Local Certificate Group

Particularité sur FortiADC, il faut créer un Local Certificate Group dans lequel on met le certificat qu'on vient d'importer. Ce groupe sera utilisé dans le profile SSL du VS par la suite.

Dans le menu System > Manage Certificates puis dans l'onglet Local Certificate Group, cliquer sur Create New

\[Capture Local Certificate Group\]

Nommez le pour qu'on le reconnaisse, puis cliquer sur Create New, Selectionnez le certificat que vous venez d'importer

\[Capture\]

\[Capture\]

### Créer un Real Server (correspond à un Node sur f5)

Comme sur un f5, pour loadbalancer vers des serveurs, il faut les définir en tant que Real Server (ou Node sur f5).

Pour cela, aller dans le menu Server Load Balance > Real Server pool puis dans l'onglet Real Server en haut de la page.

Dans notre exemple 4 Real Server ont déjà été créer sur le FortiADC.

\[Capture Nodes\]

Pour en créer un nouveau, cliquer sur Create New.

Dans le champ Name, mettre le nom du Real Server (en général le nom de la machine)

Server type: Static (on veut le faire par IP/Nom)

Status: au choix mais en général Enable

Type: IP

Address: mettre l'adresse IPv4 du Node (si vous êtes encore en IPv4)

Address6: mettre l'adresse IPv6 du Node (si vous êtes en IPv6)

Cliquer sur Save pour sauvegarder

\[Capture Conf Nodes\]

Note: le Status Enable rend le Real Server actif s'il est rajouté dans un Real Server Pool. En le mettant en Disable, il sera donc désactivé (aucun trafic n'ira vers lui). En mettant Maintain, vous laissez les sessions qui sont en cours sur ce Real Server mais aucune nouvelle ne sera envoyée vers lui ; c'est une option en générale utilisée pour faire une maintenance sur le Real Server ou de le retirer du Real Server Pool en laissant les sessions se terminer d'elles même.

### Créer un Real Server Pool (correspond à un Pool sur f5)

Dans le même menu (Server Load Balance > Real Server pool), allez dans l'onglet Real Server Pool pour créer une Pool.

\[Capture Pool\]

Dans notre exemple, il existe déjà deux Pool mais nous allons en créer une nouvelle. Pour cela, cliquer sur Create New

Name: mettre le nom de la Pool

Address Type: IPv4 ou IPv6

Type: Static

Health Check: si vous voulez que le FortiADC supérvise l'état des Real Servers il faut activer cette option et un menu apparaitra pour configurer les tests de vie.

Health Check Relationship: AND ou OR (à vous de voir si vous voulez additionner les tests de vie avec la fonction AND ou si vous préférez qu'un seul critère suffise pour déterminer l'état de santé des Real Servers).

Health Check List: selectionnez le test de vie que vous souhaitez appliquer sur le Node. Dans notre exemple on va prendre LB\_HLTHCK\_HTTPS ainsi que LB\_HLTHCK\_ICMP ; il faudra donc que le Real Server réponde au Ping ainsi qu'aux requêtes en HTTPS sur le port 443.

Direct Route Mode : on ne sait pas trop ce que ça fait, on ne va pas l'activer (je vous invite à lire la documentation officielle de Forti parce que j'y comprends rien).

Real Server SSL Profile: vous pouvez spécifier le profile SSL serveur (liaison entre le FortiADC et le Real Server Pool).

Une fois que c'est fait, enregistrez en cliquant sur Save. Vous pourrez ajouter les Real Server en tant que membre juste après.

\[Capture Pool\]

Réouvrez le Real Server Pool nouvellement créé et vous pourrez rajouter des membres en cliquant sur Create new.

Vous pourrez choisir un des Real Server que vous avez créé ou vous pourrez en créer un à la volée.

Précisez le port d'écoute du Node (ici 443). On peut mettre 0 si vous souhaitez utiliser le/les ports d'entrée du VS. Par exemple, si le VS écoute sur 80, 443 et 8080, vous devrez mettre 0 comme port de destination du Real Server dans le Real Server Pool. Toutes les requêtes qui arrivent sur le VS:80 iront sur le RealServer:80, idem VS:443 iront sur le RealServer:443 et respectivement pour 8080.

\[Capture node dans Pool\]

Voici un Real Server Pool complet:

\[Capture Pool Complet\]

### Créer un NAT Source Pool (équivalent SNAT sur f5)

Dans le menu Server Load Balance > Virtual Server puis dans l'onglet NAT Source Pool

\[Capture menu SNAT\]

Cliquer sur Create New pour créer une nouvelle NAT Source Pool.

Name: donnez un nom précédé par SNAT-

Interface: ceci doit normalement être prérempli. C'est l'interface (numéro de VLAN) associé au VDOM dans lequel vous êtes.

Address Type: IPv4/v6 au choix

Address Range... to ...: Mettre deux fois la même address car on veut SNATer avec une seule IP

\[Capture Configuration NAT Source Pool vierge\]

Cliquer sur Save pour enregistrer.

### Créer un VS

Dans le menau Server Load Balance > Virtual Server vous aurez la liste des VS existant. On peut également en créer des nouveaux en cliquant sur Create New puis selectionnant Advanced Mode

\[Capture VS\]

Une fois dans la configuration du VS, cela reste assez classique, on peut choisir le nom, le type (L7, L4 ou L2), décider d'activer/désactiver/maintain le status.

Un point d'attention: il faut choisir Layer7 pour faire du HTTPS mais sinon en Layer4 il faut choisir Full NAT pour appliquer le SNAT vers les Real Server Node.

\[Capture conf VS\]

Dans l'onglet Général, on détermine l'IP ainsi que le port d'écoute.

\[Conf Général VS\]

Dans les Resources, il faut choisir le profile HTTPS pour préciser au FortiADC qu'on veut faire de l'inspection SSL. On peut ensuite choisir le Client SSL Profile qu'on a créé précédemment avec le certificat qu'on a importé.

Dans l'onglet Security on peut appliquer des politiques de WAF Profile et AV Profile.

\[Conf Security\]

Cliquer sur Save pour créer le VS.

### Créer un VS LDAPS

Pour faire un VS LDAPS sur FortiADC, il n'existe pas de profile pé-existant, il faut choisir le profile LB\_PROF\_TCPS et mettre un certificat TLS.

### TLS1.0

Windows qui retourne “le code d'alerte irrécupérable défini par le protocole de tls est 70”

TLS1\_ALERT\_PROTOCOL\_VERSION 70

SEC\_E\_UNSUPPORTED\_FUNCTION 0x80090302

→ TLS1.0 manquant (oui ils utilisent encore ça) et à activer dans le Menu Server Load Balance → Application Resources → Profile SSL Custom et rajouter TLS1.0 (ou autre).


Exemple de profil Access Protection (WAF) pour bloquer page admin Wordpress
---------------------------------------------------------------------------
Sur wordpress, l'URI d'administration est /wp-admin/. Si l'on souhaite bloquer l'accès à cette page avec la fonction Access/Url Protection du module WAF du FortiADC alors il suffit de faire comme ceci: 

![](/fortinet/images/adc-waf-wordpress-admin-protect.png)
