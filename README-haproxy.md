# HAProxy

Le load-balancer que nous allons installer s'appelle HAProxy.

## Principe d'un load balancer

Un load-balancer (répartiteur de charge, en français) fait exactement ce que son nom indique : il va permettre de **répartir la charge (les requêtes HTTP entrantes) entre plusieurs serveurs**.

Autre avantage qu'apporte un load-balancer : le **failover** (basculement automatique en cas de panne, en français). Le load-balancer surveille l'état des serveurs en permanence : si l'un d'entre-eux tombe en panne, le load-balancer enverra les requêtes aux autres serveurs du cluster.

Pour finir, le load-balancer est nécessaire pour nous permettre d'installer un certificat HTTPS sur notre application et chiffrer ainsi les échanges entre nos visiteurs et l'application.

![loadbalancer](https://www.uhost.com/images/pub/art/art_loadbalancing.gif)

## Installation & configuration

Sur le dernier noeud du cluster (celui que nous n'avons pas joint au cluster), lancez la commande suivante pour installer HAProxy.

```bash
apk add haproxy
```

:bulb: La commande `apk add` permet d'installer des paquets sur la distribution Alpine Linux utilisée par Play-With-Docker.

Nous aurons aussi besoin de l'éditeur de texte nano, installez-le avec la commande `apk add nano`.

Il ne nous reste plus qu'à configurer HAProxy ! Pour celà, lancez la commande `nano /etc/haproxy/haproxy.cfg`.

Supprimez toutes les lignes à partir de la ligne `# main frontend which proxys to the backends`, et ajoutez les lignes ci-dessous (**pensez à remplacer les adresses IP des noeuds !**) :

```yaml
# Configure HAProxy to listen on port 80
frontend http_front
   bind *:80
   stats uri /haproxy?stats
   default_backend http_back

# Configure HAProxy to route requests to swarm nodes on port 8888
backend http_back
   balance roundrobin
   server node1 <ip_noeud_1>:8888 check
   server node2 <ip_noeud_2>:8888 check
   server node3 <ip_noeud_3>:8888 check
```

Le `frontend http_front` nous permet de configurer la "porte d'entrée" d'HAProxy : il va écouter sur le port 80 (grâce à l'instruction `bind *:80`), mettre à disposition une page de statistiques à l'adresse `/haproxy?stats` (grâce à l'instruction `stats uri ...`) et répartir la charge sur les noeuds définis dans le `backend` intitulé `http_back`.

Dans le `backend http_back`, on vient définir le mode de répartition (ici, `roundrobin`, qui est un [algorithme d'ordonnancement](https://fr.wikipedia.org/wiki/Round-robin_(informatique))) et les différents serveurs sur lesquels répartir la charge (avec les instructions `server ...`). On nomme chaque serveur (`node1`, `node2`, `node3`), on indique l'adresse IP et le port du service sur ce serveur, et l'instruction `check` indique à HAProxy qu'il doit surveiller l'état de ce serveur.

Une fois les modifications effectuées, on peut enregistrer & quitter nano avec `Ctrl+X`.

Il ne reste plus qu'à démarrer HAProxy avec la commande `haproxy -f /etc/haproxy/haproxy.cfg -d`. L'argument `-d` permet de lancer HAProxy en mode debug, une fois que tout est fonctionnel on peut le lancer sans cet argument.

## Tester !

Pour tester, ouvrez le port 80 sur le noeud du load-balancer. Vous allez arriver sur notre application. Connectez-vous, actualisez la page une fois la connexion effectuée. L'application devrait vous redemander de vous connecter ... et c'est tout à fait normal ! 

En effet, les sessions PHP qui permettent de savoir si nous sommes authentifié ou non sont stockés sur chaque conteneur du cluster. Pour que la connexion soit maintenue à travers l'ensemble des conteneurs du cluster, il faudrait mettre en place un espace de stockage partagé entre tous les conteneurs, sur lequel nous stockerions les sessions PHP.

Vous pouvez aller sur l'url `/haproxy?stats` pour avoir des statistiques sur HAProxy (l'état des différents serveurs, par exemple).

## Aller plus loin ...

L'étape suivante serait d'installer un certificat HTTPS sur notre load-balancer, or malheureusement ce n'est pas possible dans l'environnement Play-With-Docker.

> Et si le load-balancer tombe en panne ?

Nous venons de créer ce qu'on appelle un **SPOF, un Single Point Of Failure**, ou poiunt de défaillance unique en Français. En effet, si HAProxy ou le serveur sur lequel nous l'avons installé cesse de fonctionner, notre application n'est plus accessible. La solution serait d'installer non pas un mais **deux load-balancer, configurés en failover** (si l'un tombe en panne, l'autre prend le relai).
