# Notes en Vrac
192.168.1.99 est l'ip par défaut de l'interface de management d'un fortiproxy sorti du carton. On peut configurer son client sur l'ip 192.168.1.100 pour contacter la GUI ou s'y connecter en SSH

/!\\ un rollback de version d'OS fait un factory reset du boitier. Il faut réimporter la configurer (qu'on est obligé de télécharger avant le rollback de toute façon).

Pour les Explicit Web Proxy:

*   On ne peut avoir qu'un Explicit Web Proxy qui écoute sur 0.0.0.0 même si on lui affecte une interface particulière.
*   Pour en créer d'autre, il faut absolument spécifier une adresse IP d'écoute et l'affecter à une interface réseau sinon l'interface répond une erreur générique bidon.
*   L'idéal, c'est de toujours affecter une IP à un Explicit Web Proxy à chaque fois qu'on y attache une interface.

L'interface en 2.0.9 devrait être un peu revue car pas clairement explicite (c'est le cas de le dire ;))

J'ai galéré à faire une MAJ de 2.0.9 à 7.0.4 : l'interface GUI n'était plus disponible, je n'ai pas réussi à débugger la situation donc j'ai rollback

Je n'ai pas réussi à réimporter un backup depuis une clé usb, la commande execute restore config usb myfilename.conf ne trouvait aucune clé usb.

Les commandes execute usb-disk list  et execute usb-device list me retournaient aucun résultats. Je n'ai pas compris. J'ai cependant pu formater la clé usb depuis la CLI FortiProxy preuve qu'il voyait quand même que chose.

# Bug FortiProxy FTP PASSV (bug "by design" d'après le support)

Schema: Client ------> FortiProxy ------> Serveur.

Lors qu'on utilise le Fortiproxy en tant que proxy FTP et que le serveur demande de passer en mode passif (PASSV) un port passif est défini par le serveur.
Le Fortiproxy contacte bien le serveur sur le port défini mais par contre, il choisit un port random pour passer en mode passif avec le client.
En terme de flux à ouvrir, il faut donc ouvrir le port TCP/22 (FTP) ainsi que TCP/1024-65535 entre le client et le FortiProxy. Ensuite, il faut ouvrir TCP/22 et les ports passifs définis par le serveur (dans mon cas TCP/50000 à 51000).


# Configurer interface management dédié

Sur le primaire: 

Aller dans System HA > Cocher la case "Management Interface Reservation"
Sélectionner le port dédié au management
Définir la GW.

Ensuite aller dans Network Interface > Selectionner le port dédié au management et le configurer avec les IP et bonnes infos.

Pour configurer le secondaire, depuis le primaire, aller en CLI 

execute ha manage 1 <username>
config system ha
  set ha-mgmt-status ensable
  config ha-mgmt-interfaces
    edit 1
        set interface "port2" (c'est le port choisi pour l'administration)
        set gateway <gw>
    next
  end
  end
  
Une fois que c'est fait, il faut configurer le port2 (port choisi pour le management dédié)
config system interface
  edit "port2"
      set ip 10.A.B.C 255.255.255.0
      set allowaccess ping https ssh http
      set type physical
      set alias "Management-Dedie"
      set role lan
      set snmp-index 2
  next
end


# Mise à jour d'un FortiProxy de 7.0.5 à 7.2

## Prérequis: 
* Aller voir la release note: https://docs.fortinet.com/document/fortiproxy/7.2.0/release-notes/146706/introduction
* Aller sur https://support.fortinet.com/Download/FirmwareImages.aspx pour télécharger la version 7.2 du FortiProxy.
* Vérifier l'intégriter du firmware avec le hash
* Faire un backup de la configuration du FortiProxy

## Mise à jour

* Aller dans System > Firmware > Select Firmware > File Upload
* Selectionner le fichier .out de mise à jour et l'importer.
* Cliquer sur Backup Config and Upgrade. 

A partir de la, le fortiproxy fera la mise à jour.
Le système va redémarrer.
  
# FortiProxy Explicit FTPS Proxy
  ## Activer la fonction Explicit FTP Proxy sur le FortiProxy
  * Dans Proxy Settings > FTP Proxy, activer le Explicit FTP Proxy.
  * Définir les IP d'entrée et de sortie (on peut laisser 0.0.0.0 car dans la Policy on va Binder le FTP Proxy à une interface
  * Définir les Incoming ports
  
  ## Activer l'écoute du service Explicit FTP Proxy sur l'interface réseau souhaitée
  * Aller dans Network > Interfaces
  * Choisir son interface d'écoute (dans mon cas un VLAN sur un aggregat de lien) et double cliquer dessus
  * Descendre tout en bas de la page dans la section Miscellaneous et activer Explicit FTP Proxy
   _Note: à ce stade, le proxy FTP est activé mais pas encore la fonction SSL (en vrai TLS, svp ne faites plus de SSL) pour faire du FTPS_
  
  ## Activer la fonction SSL
  * en CLI: 
  
  config ftp-proxy explicit
  
  set ssl enable
  
  end
  
  ## Créer la Policy de proxification FTPS
  * Dans Policy & Object > Policy, créer une nouvelle Policy
    * Type: FTP
    * Name: Policy FTPS
    * Outgoing Interface: Spécifier l'interface de sortie du trafic une fois proxifié
    * Source: <votre source>
    * Destination: <le serveur de destination FTPS>
    * Action: ACCEPT
    * Dans Proxy Options, il va falloir spécifier le certificat à proposer au client. Ce certificat est porté par le Fortiproxy et sera proposé au client lorsqu'il se connecte. Ici j'ai laissé le certificat par défaut, mais dans un environement de production il faut faire une CSR, la faire signer par la PKI de l'entreprise et ensuite réimporter le certificat (je ferai surement une doc un jour pour ça).
    * Vous pouvez laisser le reste par défaut même si dans mon cas j'ai activé l'AV, l'IPS et je log toutes les sessions dès qu'elles démarrent.
    * Tout en bas, activer "Enable this policy".
  * Cliquer sur OK pour valider la création de la policy
  
  ![](/fortinet/images/Policy.png)
  
## Dans le cadre d'un transfert FTPS:

530 The server does not support SSL. Please reconnect with SSL disabled.

Cela veut dire que le fortiproxy n'arrive pas à joindre le serveur de destination. C'est une erreur qui porte à confusion.

## Debug problème de license / communication avec fortiguard
``` 
get system status
get system perf status
execute ping 8.8.8.8
execute ping update.fortiguard.net
execute ping service.fortiguard.net
show full system fortiguard
show full system central-management
diagnose debug rating
diagnose autoupdate version
diagnose debug crashlog read
diagnose debug application update -1
diagnose debug enable
execute update-now 
``` 
  
Let it run for 5 minutes, then stop the debug by: 
``` 
diagnose debug disable
diagnose debug reset 
``` 

Once the same is done, Please verify the status of subscription license and update us with the logs. Thank you.

  

## Liens utiles pour FortiProxy: 
  
  Utiliser un proxy pour joindre internet pour mettre à jour fortiguard (dans le cas ou le fortiproxy n'est pas directement exposé sur internet: 
  https://community.fortinet.com/t5/FortiGate/Technical-Tip-FortiGuard-updates-using-a-proxy-server/ta-p/191904

## Bugs en 7.0.7
  
  Lors de la mise à jour du fortiproxy de 7.0.4 à 7.0.7 pour corriger la CVE-2022-40684 fait sauter tous les certificats pré-configurés dans les Profils d'inspection SSL/SSH.
  
  Il faut manuellement les reconfigurer avec les bons certificats pour que ça fonctionne.
  
  **Note**: Pour le moment le profil d'inspection SSL/SSH ne fonctionne pas en FTPS: il propose uniquement le certificat autosigné du fortiproxy et pas celui configuré. Un case est ouvert chez Forti.
  
## Configuration Filezilla pour FortiProxy en FTP / FTPS Explicite / FTPS Implicite
  ### FTPS Explicite
  
  Pour le FTPS Explicite, les deux configurations ont l'air de fonctionner: le Serveur mandataire FTP ainsi que le Serveur mandataire générique
  
  ![](/fortinet/images/FTPExplicite.png)
  
  ### FTP en clair:
  
  ![](/fortinet/images/FTPetFTPSImplicite.png)
  
  ### FTPS Implicite
  
  Pour le FTPS Implicite, il ne faut pas configurer le FTP > Serveur mandataire FTP mais le Serveur mandataire générique comme ceci:
  
  ![](/fortinet/images/FTPetFTPSImplicite.png)
  
