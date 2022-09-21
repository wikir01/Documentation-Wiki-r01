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

530 The server does not support SSL. Please reconnect with SSL disabled.

Cela veut dire que le fortiproxy n'arrive pas à joindre le serveur de destination. C'est une erreur qui porte à confusion.
