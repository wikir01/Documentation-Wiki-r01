**Filtres Wireshark utiles**

Afficher tous les packets avec un code réponse HTTP 200:

```http.response.code == 200```

Afficher toutes les requêtes DNS:

```dns.flags.response == 0```

Afficher toutes les réponses DNS:

```dns.flags.response == 1```

Afficher toutes les requêtes DNS A:

```dns.qry.type == 1```

Afficher toutes les requêtes HTTP GET:

```http.request.method == "GET"```

Afficher toutes les requêtes HTTP POST:

```http.request.method == "POST"```

Afficher toutes les serveurs Apache: 

```http.server contains "Apache"```

Afficher toutes les pages .php et .html: 

```http.host matches "\.(php|html)"```

Afficher tous les packets qui utilisent les ports 80, 443 et 8080

```tcp.port in {80 443 8080}```

Faire des recherches en majuscule/minuscules dans un champ texte: 

```upper(http.server) contains "APACHE"```

```lower(http.server) contains "apache"```
