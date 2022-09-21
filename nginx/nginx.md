Dans /etc/nginx/conf.d/stream, déposer le fichier de conf nomduVS.conf (par exemple: dns.conf).

Le fichier de configuration est en deux parties: 

*   upstream: contient la liste des serveurs de destinations ainsi que le port utilisé sur ces serveurs
*   server: contient la configuration du virtual server nginx (port d'écoute du VS, les logs)

Voici un exemple de fichier de configuration pour du load balancing simple vers deux serveurs qui écoutent en TCP 53.
```
upstream VS-DNS {
         server 10.0.0.1:53;
         server 10.0.0.2:53;
}
```
`server {`

         `listen 53:`

         `listen 53 udp;`

         `proxy_pass VS-DNS;`

         `error_log /chemin/vers/fichier/error.log info;`

         `accesslog /chemin/vers/fichier/access.log.json upstreamlog_json_combined;`

Je vous renvoie vers la doc officielle de nginx pour plus de détail.
