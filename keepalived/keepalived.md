L'outil keepalived permet de mettre en place de la haute disponible sous linux (en utilisant le protocole VRRP).

Dans /etc/keepalived/keepalived.conf:
```
global_defs {
   notification_email {
     truc@email.com
   }
   notification_email_from lb1@domain.com
   smtp_server votreserveursmtp
   smtp_connect_timeout 30
}
vrrp_script check_nginx_alive {
    script “pidof nginx” #on verifie que le processus de nginx tourne toujours
    `interval 4 #on execute le script toutes les 4 secondes
    `rise 2 # 2 Success pour OK
    `fall 2 # 2 Echecs pour KO
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual__router__id 51 #mettez n'importe quel router ID qui ne soit pas déjà utilisé sur votre L2
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass MoTdEpAsSeCoMpLeXe
    }
    virtual_ipaddress {
        10.0.0.1 #adresse virtuelle partagée entre les différents acteurs de cette instance VRRP
    }
    track_script {
        check_nginx_alive
    }
}

Plus d'info ici en anglais: [https://www.redhat.com/sysadmin/keepalived-basics](https://www.redhat.com/sysadmin/keepalived-basics)
