**show interface status errdisabled** : liste les interfaces en errdisabled ainsi que la raison de l'erreur (exemple: speed-misconfigured).

**arista errdisable speed-misconfigured** : Sur les arista 7060SX2, le "fond de panier" se fait par groupe de 4 ports. Par exemple, et1 à et4 partagent le même fond de panier (idem et5 à 8, et45 à 48 etc..). Le changement de la vitesse d'un des 4 ports implique le changement de vitesse des 3 autres. Par exemple, si le premier port est configuré en 10Gbps, alors les 3 autres ports seront également forcés en 10Gbps. Il est courant d'avoir l'erreur **speed-misconfigured** si le premier port a un speed forcé alors que les 3 autres non. Les trois autres ports auront donc l'erreur **errdisabled speed-misconfigured**. Il faut donc forcer l'intégralité des 4 ports du groupe à la même vitesse.

## capture de packet sur Arista
Cette procédure recense la méthode pour réaliser une capture réseau (style pcap) sur les switches Arista


    Se connecter en CLI sur le switch concerné
    Passer en mode config

    Switch#conf t

    Switch(config)#

    Créer une session de monitoring qui utilisera le CPU en destination
    Cela permet de rediriger le trafic choisi vers le CPU

    Switch(config)#monitor session exempletcpdump destination cpu

    Switch(config)#

    identifier la source à rediriger

    Switch(config)#monitor session exempletcpdump source ethernet 10

    Vérifier la création de cette session de monitoring

    Switch#

    Switch#show monitor session

    Session exempletcpdump

    ------------------------

    Source Ports:

      Both:        Et10

    Destination Ports:

        Cpu :  active (mirror3)

    Switch#

    Activer le tcpdump sur cette session

    Switch#tcpdump monitor exempletcpdump


    Il est ensuite possible d'utiliser plusieurs options :
        FILE → permet de générer un fichier exportable
        FILTER → limiter la capture à un filtre défini (ici exemple d'un filtre sur le vlan 358)
    Il est ensuite possible d'extraire ce fichier enregistré dans le dossier flash en passant MobaXterm ou winscp par exemple
    Ce fichier est directement utilisable par Wireshark pour une analyse plus fine

    Switch#tcpdump monitor exempletcpdump file flash:exempletcpdump filter vlan 358

    tcpdump: WARNING: mirror3: no IPv4 address assigned

    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode

    listening on mirror3, link-type EN10MB (Ethernet), capture size 65535 bytes

    11:37:38.357233 00:00:0c:9f:f1:65 > 01:00:5e:00:00:66, ethertype 802.1Q (0x8100), length 118: vlan 358, p 0, ethertype IPv4, 172.19.27.35.hsrp > 224.0.0.102.hsrp: HSRPv1

    11:37:38.567306 2c:31:24:b9:80:62 > 01:00:5e:00:00:66, ethertype 802.1Q (0x8100), length 64: vlan 358, p 0, ethertype IPv4, 172.19.27.34.hsrp > 224.0.0.102.hsrp: HSRPv2

    11:37:39.115790 2c:31:24:b9:80:62 > 01:00:5e:00:00:66, ethertype 802.1Q (0x8100), length 118: vlan 358, p 0, ethertype IPv4, 172.19.27.34.hsrp > 224.0.0.102.hsrp: HSRPv1

    11:37:39.177044 2c:31:24:b9:80:0a > 01:00:0c:cc:cc:cd, ethertype 802.1Q (0x8100), length 68: vlan 358, p 0, LLC, dsap SNAP (0xaa) Individual, ssap SNAP (0xaa) Command, ctrl 0x03: oui Cisco (0x00000c), pid PVST (0x010b): STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 1166.2c:31:24:b9:80:00.800a, length 42

    11:37:41.031191 00:00:0c:9f:f1:65 > 01:00:5e:00:00:66, ethertype 802.1Q (0x8100), length 118: vlan 358, p 0, ethertype IPv4, 172.19.27.35.hsrp > 224.0.0.102.hsrp: HSRPv1

    11:37:41.178243 2c:31:24:b9:80:0a > 01:00:0c:cc:cc:cd, ethertype 802.1Q (0x8100), length 68: vlan 358, p 0, LLC, dsap SNAP (0xaa) Individual, ssap SNAP (0xaa) Command, ctrl 0x03: oui Cisco (0x00000c), pid PVST (0x010b): STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 1166.2c:31:24:b9:80:00.800a, length 42

    11:37:41.596212 2c:31:24:b9:80:62 > 01:00:5e:00:00:66, ethertype 802.1Q (0x8100), length 118: vlan 358, p 0, ethertype IPv4, 172.19.27.34.hsrp > 224.0.0.102.hsrp: HSRPv1

    ^C

    7 packets captured

    44 packets received by filter

    0 packets dropped by kernel

    Switch#


