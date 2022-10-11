# Documentation Stormshield sorti du carton

Les paramètres à utiliser sur des logiciels comme Putty, minicom ou d’autres terminaux sont les suivants :

115200 bauds, 8 bits de données N pour pas de parité, 1 bit de stop (ou 115200/8N1), pour les modèles SN160(W), SN210(W) et SN310(W), SN510, SN710, SN2100, SN3100 et SN6100.

9600/8N1, pour le modèle SN910.

- compte par défaut: admin/admin

- Se connecter sur le port 1 en RJ45 

Par défaut, le stormshield propose un DHCP en 10.0.0.10 au client et le stormshield écoute en 10.0.0.254

![](/stormshield/images/ConfIPWindows.png)

![](/stormshield/images/puttystormshield.png)

**portinfo** permet d'avoir le status des ports du stormshield.

![](/stormshield/images/portinfo.png)

L'URL d'administartion est la suivante https://10.0.0.254/admin

Compte: admin/admin

![](/stormshield/images/pageaccueilstormshield.png)

![](/stormshield/images/Accueilstormshieldauthentifié.png)

Vous verrez un message indiquant qu'il faut lui donner une licence. Vous retrouverez la licence sur le site mystormshield.eu.

Allez dans Système > Licence et installer le fichier de licence. 

## Mettre à jour un Stormshield

Aller dans Configuration > Système Maintenance et y uploader la mise à jour.

Lors de la mise à jour, il est possible d'utiliser la partition de sauvegarde si besoin de revenir en arrière (recommandé).

Dans le cadre d'un cluster en HA, commencer par le secondaire, vérifier que ça fonctionne en basculant sur le secondaire qui est à jour puis si ça marche faire la mise à jour sur le primaire.

## Mise en HA d'un cluster de Stormshield SN710

Pour identifier le port sur lequel la HA va être montée, il faut aller en console et faire la commande **portinfo**.

Capture portinfo-master.

Le câble ethernet pour la HA est branchée sur le port 7 du boitier qui correspond au port dmz5 dans le système stormshield /!\ Le port dmzX ne correspond souvent pas au numéro du port physique sur l'appliance /!\

Il fait répéter l'action sur le SN710 secondaire pour aussi identifier le port système idbX. 

Brancher le câble console sur le secondaire, se connecter avec admin/admin et faire **portinfo**

Capture portinfo-secondaire

On peut voir (qu'avec de la chance) le port 7 du S710 secondaire s'appelle aussi dmz5.

Répéter l'action pour faire une HA sur deux ports différents de chaque boitier (objectif: avoir deux câbles ethernet entre les deux SN710 pour la Haute Dispo)

Dans mon cas, c'est dmz6 qui correspond au port physique 8 de mes deux stormshield.

Nous avons donc dmz5 qui correspond aux ports physiques 7 et idb6 au ports physiques 8

Pour construire la HA, aller dans Configuration > Système > Haute Disponibilité 

Définir les interfaces de HA avec les informations précédemment récupérées.

Capture ConfigHA

L'assitant va vous demander de rentrer la clé prépartagée pour la mise en cluster. (Eviter les " dans le mot de passe).
Cocher la case "Chiffrer la communication entre les firewalls"

Capture configHA2

Faites suivant.
Vous aurez un résumé de la configuration avant de la valider.

Capture configHA3

Cliquer sur Terminer. A ce moment, le stormshield primaire est configuré et il faut faire la même configuration sur le secondaire.

Capture configHA4 et configHA5

Sur le secondaire, aller Configuration > Système > Haute Disponibilité et faire **Rejoindre un groupe de firewalls (cluster) existant**

Entrer les informations nécessaires. Préciser l'IP du membre primaire: 1.1.1.1 ainsi que la clé PSK définie au préalable. 

En cliquant sur suivant vous pourrez confirmer tous les paramètres déjà entrés et valider la mise en cluster. 

Les stormshield vont redemarrer (et feront un petit bruit mignon)

Une fois que les deux stormshields ont redemarré, en façade, il y a 3 LED: la première LED est le status du cluster (si le membre est primaire ou secondaire).

LED Fixe = primaire

LED clignotante = secondaire 

Une interface de type **interne (protégée)** ne doit voir que des IP du réseau qui est déclaré sur l'interface. Sinon c'est considéré comme une usurpation d'IP. 

Une interface de type **externe (publique)** peut voir des IP autres que celles du réseau sur lequel elle est (c'est fait pour une interface exposée à internet / le reste du monde).

Dans Configuration > Réseau > Interfaces, il faut sortir les interfaces qu'on souhaite utiliser (en dehors de la HA) du bridge. Le bridge c'est la conf par défaut et dans mon cas je n'en ai pas besoin. Je sors donc deux interfaces: une de management et une pour les flux de production. 

## Dédier une interface de management dédiée 

Aller dans Configuration > Réseau > Interfaces et sélectionner une interface qui sera dédiée au management. La configurer comme ceci:

Capture Configuration Management.png

### Limiter les accès à l'interface de management

**Attention** Par défaut, le Stormshield propose son interface d'administration sur tous les ports. Il faut limiter l'accès à l'interface de management au réseau d'administration. 

Pour cela, aller dans Configuration > Système > Configuration > Administration du Firewall et rajouter la plage IP autorisée à accéder à l'interface de management.

Il faut créer autant d'objets dans Configuration > Objets que de plages IP qu'on veut autoriser à joindre le management.

## Configurer une interface physique comme un trunk avec plusieurs VLANs

Piège: il faut mettre l'interface physique en DHCP avant de mettre des VLAN dessus.

Une fois que l'interface physique est configurée en DHCP (même si on n'en fait pas..) il faut maintenant créer les VLAN à rattracher sur cette interface. Toujours dans le même menu Configuration > Réseau > Interfaces, il faut cliquer sur "Ajouter" et sélectionner **Ajouter un VLAN**

Capture Création de VLAN



