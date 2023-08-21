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
  
  set ssl-cert "nomducertificat.cer" /!\ j'ai l'impression que c'est super important car le CA Certificate inclus dans le profile ne fonctionne pas en v7.2.4
  
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
  
  **Note en 7.2.4**: J'ai le même problème. Les profiles d'inspection SSL/SSH ne fonctionnent pas pour du FTPS. J'ai du faire ceci en CLI : «set ssl-cert “nomducertificat.cer”». Le support de Forti ne m'a pas aidé après plein de prises de traces et de logs et d'échanges de configuration. 
  
## Configuration Filezilla pour FortiProxy en FTP / FTPS Explicite / FTPS Implicite
  ### FTPS Explicite
  
  Pour le FTPS Explicite, les deux configurations ont l'air de fonctionner: le Serveur mandataire FTP ainsi que le Serveur mandataire générique
  
  ![](/fortinet/images/FTPExplicite.png)
  
  ### FTP en clair:
  
  ![](/fortinet/images/FTPetFTPSImplicite.png)
  
  ### FTPS Implicite
  
  Pour le FTPS Implicite, il ne faut pas configurer le FTP > Serveur mandataire FTP mais le Serveur mandataire générique comme ceci:
  
  ![](/fortinet/images/FTPetFTPSImplicite.png)
  
## Débug de proxy forward
  
    Cette commande donne le status de santé du module WAD du fortiproxy pour les serveurs proxy forward. On peut y voir l'état de santé des proxy forward.
  
   ``` 
  diag wad webproxy forward-server
   ``` 
  ``` 
VDOM=root server_name=proxy_forward_1
        addr=ip/163.116.128.81:8080 health_check=disable down-opt=block 
        conns: succ=4 fail=0 ongoing=2 hits=4 blocked=0 
        monitor: succ=0 fail=0 
        num_worker_load=55 state=up psv_tm=588(sec) arbiter_tm=2(sec) 

VDOM=root server_name=proxy_forward_2
        addr=ip/163.116.128.80:8080 health_check=disable down-opt=block 
        conns: succ=9 fail=2 ongoing=2 hits=11 blocked=0 
        monitor: succ=0 fail=0 
        num_worker_load=55 state=try_once psv_tm=700(sec) arbiter_tm=10(sec)

  ``` 
  ``` 
  diag wad webproxy forward-server-group
  ``` 

  On peut voir l'état de santé du groupe avec cette commande.

  ## Utilisateur qui a "Group Information Query Failed"
  
  ![](/fortinet/images/Group_information_Query_Failed.png)

```
diag debug reset
diag debug console timestamp enable
show auth rule
show auth scheme
show user group
diagnose wad filter src x.x.x.x --> Filtre sur l'IP du client
diagnose wad debug enable category policy
diagnose wad debug enable category auth
diag wad debug enable level verbose
Diag debug app fnbamd -1
diagnose debug enable 
```

### Exemple et analyse de traces
```
2023-08-18 09:50:29 [1718] fnbamd_ldap_init-search filter is: (&(userPrincipalName=julien@DOMAIN.R01.FR)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))
2023-08-18 09:50:29 [1728] fnbamd_ldap_init-search base is: DC=DOMAIN,DC=R01,DC=FR
...
2023-08-18 09:50:29 [750] fnbamd_ldap_build_dn_search_req-base:'DC=DOMAIN,DC=R01,DC=FR' filter:(&(userPrincipalName=julien@DOMAIN.R01.FR)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))
...
2023-08-18 09:50:29 [1244] __fnbamd_ldap_dn_next-No DN is found.
2023-08-18 09:50:29 [1053] __ldap_rxtx-Change state to 'Done'
2023-08-18 09:50:29 [724] __ldap_stop-Conn with AA.BB.CC.DD destroyed. (IP de l'AD)
2023-08-18 09:50:29 [216] fnbamd_comm_send_result-Sending result 1 (nid 0) for req 1098542827, len=2148
[E]2023-08-18 09:50:29.684943 [p:12765] wad_group_info_auth_on_fnbam_resp :142 auth resp:0x7ffd809c3140 ,auth failure auth result:9
[I]2023-08-18 09:50:29.684959 [p:12765][s:2020264253][r:172993228] wad_http_auth_status_proc :10661 ses_ctx: ses_ctx:cx|Phx|Me|Hh|C|A7m|O authenticate result=group-query-failed
```

### Explications
Le processus **wad** détecte qu'il y a une authentification / membership lookup à faire et demande au processus **fnbamd** de le faire
**wad** process will basically see that there is an authentication/membership lookup to be done and ask the respective process **fnbamd** to do it.

Le processus **fnbamd** essaie de gérer l'authentification / vérifier l'appartenance aux groupes avec les paramètres que **wad** lui a donné (entre autre, le userPrincipalName)
**fnbamd** tries with the respective settings and the username supplied.

Note: userPrincipalName=julien@DOMAIN.R01.FR et le compte associé doivent exister et être actif dans l'AD

**fnbamd** asks for an Attribute name **UserPrincipalName** with the value julien@DOMAIN.R01.FR below the tree DC=DOMAIN,DC=R01,DC=FR and the LDAP server returns that it doesn't have a result for this query. Literally that this Attribute UserPrincipalName with value julien@DOMAIN.R01.FR does not exist.

It likewise returns this information to wad, whereas wad the says that the query it made to fnbamd returned an error. Likewise, it says that the group information query failed.

Dans mon cas, le compte julien@DOMAIN.R01.FR a été créé avec un oubli de lettre : userPrincipalName=jlien@DOMAIN.R01.FR alors que le CN et le DisplayName étaient bons.

__Conclusion__ Il faut vérifier dans l'AD les paramètres du compte car le userPrincipalName ne correspond pas au jeton Kerberos.
