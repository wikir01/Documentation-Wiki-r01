# Supprimer l'overlay de NSX-T 
_note: uniquement dans le cas où cela a été installé par erreur et qu'on souhaite faire que de la microsegementation._

## Dans vCenter :
- migrer les cartes réseau des EDGE sur un réseau vmnetwork (réseau poubelle).

## Dans Networking :
- Supprimer les Tier1-gateway
- Supprimer les Tier0-gateway : supprimer la VIP (et enregistrer) puis supprimer supprimer les interfaces de la Tier-0 Gateway.
- Supprimer les segments uplinks des VM edge

## Dans System: Edge cluster : 
- Retirer les EDGE VM du cluster
- Supprimer les edge cluster (rafraichir la page car pas de bouton refresh)

## Dans System > Fabric > Nodes : 
- Cocher les EDGE VM et supprimer. Cela décomissionne les VM EDGE dans le vcenter.

## Dans System > Fabric > Transport Zone: 
- Supprimer la TZ Uplink
- Supprimer la TZ Overlay
