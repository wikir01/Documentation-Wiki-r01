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

Dans le cadre d'un transfert FTPS:

530 The server does not support SSL. Please reconnect with SSL disabled.

Cela veut dire que le fortiproxy n'arrive pas à joindre le serveur de destination. C'est une erreur qui porte à confusion.

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
- Aller voir la release note: https://docs.fortinet.com/document/fortiproxy/7.2.0/release-notes/146706/introduction
- Aller sur https://support.fortinet.com/Download/FirmwareImages.aspx pour télécharger la version 7.2 du FortiProxy.
- Vérifier l'intégriter du firmware avec le hash
- Faire un backup de la configuration du FortiProxy

## Mise à jour

Aller dans System > Firmware > Select Firmware > File Upload
Selectionner le fichier .out de mise à jour et l'importer.
Cliquer sur Backup Config and Upgrade. 

A partir de la, le fortiproxy fera la mise à jour.
Le système va redémarrer.

## Liens utiles pour FortiProxy: 
  
  Utiliser un proxy pour joindre internet pour mettre à jour fortiguard (dans le cas ou le fortiproxy n'est pas directement exposé sur internet: 
  https://community.fortinet.com/t5/FortiGate/Technical-Tip-FortiGuard-updates-using-a-proxy-server/ta-p/191904


