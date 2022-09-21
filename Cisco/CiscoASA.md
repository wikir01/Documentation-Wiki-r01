 comment sniffer un flux (façon tcpdump)
========================================

\[ peut surcharger le proc, à éviter \]

    ASA# capture [nom_cature] interface inside match ip 192.168.10.10 255.255.255.255 203.0.113.3 255.255.255.255
    ASA# capture [nom_cature2] interface outside match ip 192.168.10.10 255.255.255.255 203.0.113.3 255.255.255.255
    ASA# no capture [nom_cature] interface inside
    ASA# no capture [nom_cature2] interface outside  

[https://www.cisco.com/c/fr\_ca/support/docs/security/asa-5500-x-series-next-generation-firewalls/118097-configure-asa-00.html](https://www.cisco.com/c/fr_ca/support/docs/security/asa-5500-x-series-next-generation-firewalls/118097-configure-asa-00.html)

comment afficher les logs en live et greper
===========================================

    conf t
    logging monitor debugging ( ou autre niveau )
    terminal monitor
    terminal no monitor

comment afficher et greper la table arp  
=========================================

    show arp

comment afficher les interfaces et leur état
============================================

    ASA# show interface ?
    ex: ASA# show interface ip brief | inc [recherche]
    ex: ASA# show ip | inc [recherche]

comment afficher les routes
===========================

    ASA# show route
    ASA# show route | inc [recherche]

comment afficher les sessions VPN
=================================

    ASA# show crypto isakmp sa

comment afficher le HA
======================

    ASA# show failover state
