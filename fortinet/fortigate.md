Capture de packet
=================

    diagnose sniffer packet <interface> <filtre> 6 0 a
    
    comment sniffer un flux (façon tcpdump)
    config vdom (si vdom)
    edit [nomvdom] (si vdom)
    diagnose sniffer packet any 'host 1.2.3.4 and host 3.4.5.6 and port 8080 and net 10.1.0.0/16' 4

Debug VPN
=========

clear des filtres
-----------------

    diagnose debug disable
    diagnose debug reset
    diagnose vpn ike log-filter clear

création des filtres
--------------------

    diagnose debug application ike 255 (obligatoire, fait apparaitre le traffic ike 255)
    diagnose vpn ike log-filter src-addr4 x.x.x.x --> Filtre sur @IP source (facultatif, il vaut mieux extraire des logs et CTRL+F)
    diagnose vpn ike log-filter dst-addr4 x.x.x.x  --> (facultatif)

activation du debug
-------------------

    diagnose debug enable

desactivation du debug (a taper pendant que ça déroule)
-------------------------------------------------------

    diagnose debug disable

Ensuite : Copier/Coller dans un notepad et CTRL+F du nom du VPN pour voir ce qu'il se passe.

Si le mot clé "malformed" apparait, il s'agit surement de session fantome.

Autres commandes
================

Afficher et filtrer table ARP
-----------------------------

    diagnose ip arp list | grep 1.2.3.4

Afficher interfaces et leur état
--------------------------------

    config system interface
     show (affiche les interface et les reglages)
     get (affiche les interfaces et leur état)
    end

Afficher les routes
-------------------

    config router static
     show
    end

Afficher status Haute Dispo
---------------------------

    diagnose sys ha status

Autre
-----

[https://www.brg.ch/fortigate-troubleshooting-cli-gen-en/](https://www.brg.ch/fortigate-troubleshooting-cli-gen-en/)

ip-pool = snat
vip = dnat

## Diff de conf dans un cluster en HA

[https://community.fortinet.com/t5/FortiGate/Troubleshooting-Tip-HA-synchronization-issue-cluster-out-of-sync/ta-p/193422](https://community.fortinet.com/t5/FortiGate/Troubleshooting-Tip-HA-synchronization-issue-cluster-out-of-sync/ta-p/193422)
