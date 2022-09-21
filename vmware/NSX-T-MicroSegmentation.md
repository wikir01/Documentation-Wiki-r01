NSX-T Micro Segmentation et bonnes pratiques
============================================

Article en cours de rédaction 

#### 1\. Identifier les flux de management et d’administration

Les questions à se poser:

Qui manage le service

Comment on manage le service (rdp, ssh, api ?)

D’où c’est administré (bastion d’admin ?)

#### 2\. Identifier les services communs

Quelques exemples:

DC

NTP

Exchange

Outils d’automatisation

Sharepoint

Logging

Backup

#### 3\. Identifier les environnements  

Prod

Qual

Dev

DRSF

DMZ

#### 4\. Identifier les computes Workloads

Appli developpées maison ?

Appli publiques

BDD

Est-ce qu’elles sont critiques

e. Combien de serveurs par application

#### 5\. Identifier les flux de communication entre les services

a. C’est la partie la plus dure

b. Vrealize network Insight est très bien pour ça (il donne les flux entre les machines, les ports, etc..)

c. vRealize Log Insight

d. Netflow collector

e. Documentation officielle du constructeur  

#### 6\. Developper un système de tag standard / méthodes de regroupement

##### Utiliser KEY-Value  

iTAG-SCOPE : Environnement-PROD, Service-DOMAIN, Service-PKI, Service-NTP, Critical

![](/nsx-t/6keyvalue.png)

##### Organisation standardisée d’un point de vue architecture

i. Environnement : PROD QUAL DEV PCI DMZ

![](/nsx-t/6prodqualdev1.png)

![](/nsx-t/6prodqualdev2.png)

![](/nsx-t/6prodqualdev3.png)

v. Il faut pouvoir réutiliser les tag. Tagger gros, tagger petit, mettre des tags partout et plusieurs tag. Environnement PROD vers TAG AD en LDAP.

c. Démarrer large et ensuite terminer précisement

##### Pièges :  

1\. se concentrer sur des workloads et pas sur les services communs ou environnements. Il ne faut pas démarrer par une application très précise. Il faut commencer par les services communs et ensuite mettre les règles niveau application. 1- service communs 2- détailler l’application.  

2\. Ne pas utiliser les key-value pairs

3\. Tag trop précis qui ne peuvent pas être réutilisés dans différentes politiques/groupes

4\. Aucune standardisation  

#### 7\. Implémenter les tags

Dans NSX-T 3.0, il a un nouveau menu dans Inventory > Tags qui permet de voir tous les tags qui ont été créés.

![](/nsx-t/menutag.png)

#### 8\. Groupes les objets avec les tags (statiquement ou dynamiquement)

Points importants sur le Groupage:

*   NSX-T supporte le groupage dynamique ou statique
*   KISS: Keep it stupid simple
*   Les groupes doivent reposer sur des critères précis
*   Les groupes sont utilisés dans les Rules et Policies du Distributed Firewall Group

##### Dynamic Grouping:

![](/nsx-t/8a.png)

##### Static Grouping:

![](/nsx-t/8b.png)

Il faut appliquer des TAG sur des VM et ensuite il faut les grouper. Un TAG seul ne suffit pas.

Il faut aller dans Inventory > Groups pour avoir la liste des groupes et en créer de nouveaux.

![](/nsx-t/9d.png)

![](/nsx-t/9e.png)

![](/nsx-t/9g.png)

![](/nsx-t/9f.png)

On peut voir que le groupe dispose d'un Criteria dans le Compute Members. On peut rajouter autant de Criteria qu'on veut.

![](/nsx-t/groups.png)

#### 9\. Créer les fw policy dans le DFW

*   Une DFW définit la portée d’une règle qui  se trouve dans la policy. On définit la policy en premier avec sa portée et ensuite on applique la rule de sécurité qui est appliquée sur la policy. Par exemple, on applique ça sur le groupe de l’environnement de PRODUCTION. C’est dans le champ « applied to ».  
*   POLICY AND RULE (fonction ET).

![](/nsx-t/9b.png)

Dans la capture ci dessus:

*    Deux Policies:
    *   **Admin Access** appliquée à la population **Admin Users** qui leur permet de **RDP** sur les **DC**
    *   **Domain Controller** appliquée permet aux **Domain Controllers** d'être consommés sur les services **Active Directory** (DNS, LDAP/LDAPS, etc..) et le **Reject** à la fin pour le reste

Rappel: Les règles de FW dépendent de groupes. Il faut mettre des membres dans les groupes basés sur les tags.

#### 10\. Créer les règles de FW

Astuce: Faire une EMERGENCY Rule qui allow tout sur le tag « critique » et la mettre en disable. L’activer si on foire la configuration des règles de FW.

*   Les règles vont de gauche à droite, ethernet, emergency, infra, environement et application.  
*   Les Tag sont des métadonnées qui s’appliquent sur des workload, IP, équipement physique, un segment, une plage d’IP. C’est ce que NSX utilise des objets entre eux pour y appliquer des policy.

##### Exemple de création de Policy et de Firewall pour du NTP:

Aller dans Security > Distributed Firewall, puis cliquer dans l'onglet Infrastructure:

Premièrement il faut créer une nouvelle Policy :

![](/nsx-t/10-1.png)

La nommer “NTP”

Appliquer cette policy à un groupe (Choisissez en un existant ou en créer un nouveau basé sur un criteria (tag, IP, MAC)).

Nouveau Groupe: 

![](/nsx-t/10-2.png)

Sélectionner le groupe nouvellement créé: 

![](/nsx-t/10-3.png)

Vue de la nouvelle Policy: 

![](/nsx-t/10-4.png)

Une fois la policy créée, il faut ajouter une Rule en dessous en faisait un clic droit dessus.

Donner une description dans “name” puis configurer la Rule.

![](/nsx-t/10-5.png)

La colonne **Service** correspond au L4 alors que la colonne **Profiles** correspond au L7.

Il est vraiment important de commencer par des flux Infrastructure et ensuite passer aux règles Environment et Application.

Il faut appliquer les Policy qu'a des groupes. Si on prend le DFW alors ça va s'appliquer sur TOUS les hyperviseurs alors qu'en appliquant aux Groupes cela va applique la règle qu'aux Hyperviseurs qui portent les objets du groupe en question.

Dans la catégorie **Environment**, on peut appliquer des règles d'isolation entre tenant/environnement par exemples l’environnement de **Qualification** n'a pas le droit de contacter et d'être contacter par  l’environnement de **Production.**

![](/nsx-t/tenant.png)

Les composants Infrastructure commun aux environnements ne doivent pas tagués dans un environnement..

Les flux de l'onglet Infrastructure seront transverses aux environnements. Par contre les environnements ne peuvent pas se parler.

Ensuite on passe aux flux Applicatifs dans l'onglets **Application.**

De la même manière, il faut créer une Policy et l'applique sur un groupe qui correspond à l'application. En général les VM qui portent l'application.

Ensuite on peut créer des rules:

*   “Any to Web” avec comme groupe de destination les VM (groupe Web-tier) qui portent la webapp sur le ports HTTP/HTTPS et appliqué au groupe Web-Tier (uniquement le Web-tier car c'est très spécifique comme flux qui s'applique que sur le Web-Tier).
*   “Web to App” avec comme groupe source Web-tier et comme destination App-tier sur les ports de l'application en TCP/443 appliqué à tous le DFW
*   “App to DB” avec comme groupe source App-tier vers DB-tier sur les ports des bases de données TCP/3306 appliqué à tous le DFW
*   Dernière règle a ajouter est le Drop appliqué sur le DFW

![](/nsx-t/dfw.png)

#### Exemple de micro segmentation:

![](https://content.learningplatform.vmware.com/files/file/toc_6109d63d7f3c2006a6d8ac12_2135305344---2680ffdb-a42a-454c-b117-d19bb6bedeed.png)

![](https://content.learningplatform.vmware.com/files/file/toc_6109d63d7f3c2006a6d8ac12_920237925---2e659845-e912-4307-a6da-71c00fcbb906.png)

Note: Dans la colonne **Services** de la règle ID **5097**, il aurait fallut mettre **https** et non pas **any**. Sinon les clients pourraient accéder à web-01a sur tous les ports TCP/UDP ainsi que d'autres protocoles comme l'ICMP.

![](/nsx-t/infra_nsx_microseg.png)

![](/nsx-t/app_nsx_microseg.png)
