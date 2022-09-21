Palo Alto (en admin sur le boitier) :

Prendre une capture de packet
=============================

Version courte
--------------

    tcpdump filter "host 10.10.10.5 and not port 22"

[https://live.paloaltonetworks.com/t5/Learning-Articles/How-to-Run-a-Packet-Capture/ta-p/62390](https://live.paloaltonetworks.com/t5/Learning-Articles/How-to-Run-a-Packet-Capture/ta-p/62390)

Version longue
--------------

### Créer des filtres sur les packets

           debug dataplane packet-diag set filter match source <IP_1> destination <IP_2>
           debug dataplane packet-diag set filter on
           debug dataplane packet-diag show setting

Note: Up to 4 match criteria can be configured for a given packet capture. If no source or destination IP address is specified, then "any" (0.0.0.0) is assumed.

### Configurer les captures et définir les fichiers de sorti. 

           debug dataplane packet-diag set capture stage transmit file <filename_transmit>
           debug dataplane packet-diag set capture stage receive file <filename_receive>
           debug dataplane packet-diag set capture stage firewall file <filename_firewall>
           debug dataplane packet-diag set capture stage drop file <filename_drop>

### Démarrer la capture de packet

           debug dataplane packet-diag set capture on

Note: Avant de démarrer les captures, assez vous que les filtres sont bien configuré et que le filtrage est bien activé. Par exemple :

    admin@PAN-FW> debug dataplane packet-diag show setting

Lire un pcap en CLI : 
======================

    view-pcap mgmt-pcap mgmt.pcap

Afficher et filtrer la table arp  
==================================

    show arp all | match 10.10.10.5

Afficher les interfaces
=======================

    show interfaces all

plus de details avec : 

    show interface ethernet1/20

comment afficher les routes
===========================

    show routing route

comment afficher le HA
======================

    show high-availability state
