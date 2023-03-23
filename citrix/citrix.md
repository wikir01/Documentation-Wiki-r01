## Configuration export de log syslog

Aller dans System > Auditing > Syslog Auditing > Onglet server

Créer un nouveau server cliquer sur Add
- Définir le nom du serveur
- Server Type: Server IP
- IP Address: rentrer l'IP
- Port: Port
- Logs level: définir ce qu'il vous plait (dans mon cas j'ai mis custom et j'ai tout désactivé sauf DEBUG qui génère beaucoup de log)
- Date Format: DDMMYYYY car on n'est pas des barbares 
- Time Zone: GMT
- Net Profile: il faudrait ici créer un objet qui correspond à l'IP/interface depuis laquelle les logs seront exportés. Si inexistant, créer l'objet (rentrer nom, IP, etc..)
- Transport Type: privilégier TCP sur UDP mais à vous de voir votre collecteur syslog
- Transport Profile (si TCP): choisir le profile TCP par défaut, il fait très bien le job
Cliquer sur Create pour créer le serveur.

Dans l'onglet Policy, créer une policy en cliquant sur Add
- Définir le nom de la policy de log 
- Expression Type: Advanced Policy
- Server: Sélectionner le serveur nouvellement créé
- Cliquer sur Create pour créer la policy.

A ce stable, une policy est créé et un serveur est créé mais ils ne sont pas liés. 
Pour cela, il faut cocher la case de la policy nouvellement créée > Select Action > Advances Policy Global Binding > Add Binding
Dans la fênetre "Policy Binding" qui s'ouvre, reselectionner la policy nouvellement créée et dans Global Bind Type selectionner SYSTEM_GLOBAL. Terminer avec Done.

Sur Citrix Netscaler ADC il faut cliquer sur la petite disquette en haut à droite pour sauvegarder la configuration. 
