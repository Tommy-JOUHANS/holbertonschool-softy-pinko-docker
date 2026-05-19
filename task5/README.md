# Softy Pinko Docker - Task 5 : Serveur Proxy (Reverse Proxy)

## Objectif

Placer un **serveur proxy Nginx** devant le front-end et le back-end. Le client (navigateur) ne s'adresse **plus directement** aux deux services : il communique uniquement avec le proxy, qui se charge de **router** chaque requête vers le bon service interne en fonction de l'URL.

C'est le pattern **reverse proxy**, fondamental dans toutes les architectures web modernes.

## Pourquoi un reverse proxy ?

Dans les tâches précédentes, le navigateur devait connaître **deux adresses** :

- `http://localhost:9000` pour le site (Nginx front-end)
- `http://localhost:5252` pour l'API (Flask back-end)

Cela pose plusieurs problèmes :

- **Couplage fort** : si on change le port de l'API, il faut modifier le code JavaScript de tous les clients
- **Multiplication des points d'entrée** : chaque service expose son port à l'extérieur
- **Surface d'attaque accrue** : plus de ports ouverts = plus de risques
- **CORS obligatoire** : front-end et back-end étant sur des origines différentes, il fallait `flask-cors`

Avec un reverse proxy :

- **Un seul point d'entrée public** : `http://localhost` (port 80)
- Les clients **ne savent pas** que le site et l'API sont deux services distincts
- Les services internes ne sont **pas exposés** à l'extérieur
- Plus de problème de CORS : front-end et API sont vus comme la **même origine**
- On peut ajouter du **load balancing**, du **cache**, du **HTTPS**, sans toucher au code applicatif

## Énoncé

Faire une copie du dossier `task4` et la renommer `task5`. Puis :

- Créer un dossier `proxy` dans `task5`
- Y placer un `Dockerfile` basé sur la dernière version de Nginx
- Y placer un `proxy.conf` qui :
  - écoute sur le port `80`
  - route `/` vers `http://front-end:9000`
  - route `/api` vers `http://back-end:5252`
- Copier `proxy.conf` vers `/etc/nginx/conf.d/default.conf` dans l'image
- Modifier `index.html` pour que l'AJAX appelle `/api/hello` (relatif) au lieu de `http://localhost:5252/api/hello`
- Mettre à jour `docker-compose.yml` :
  - ajouter le service `proxy` avec mapping `80:80` et `depends_on` sur les deux autres
  - **supprimer** le mapping de ports pour `front-end` et `back-end` (ils ne doivent plus être joignables depuis l'extérieur)

## Structure du projet

```
softy-pinko-docker/
└── task5/
    ├── docker-compose.yml          # modifié (ajout du proxy)
    ├── back-end/
    │   ├── api.py
    │   ├── requirements.txt
    │   └── Dockerfile
    ├── front-end/
    │   ├── softy-pinko-front-end/
    │   │   └── index.html          # modifié (URL AJAX relative)
    │   ├── Dockerfile
    │   └── softy-pinko-front-end.conf
    └── proxy/                      # nouveau
        ├── Dockerfile
        └── proxy.conf
```

## Le serveur proxy

### `proxy/Dockerfile`

```dockerfile
FROM nginx:latest

COPY proxy.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
```

Très simple : on part de l'image officielle Nginx, on remplace sa configuration par défaut par la nôtre, et on documente le port `80`.

### `proxy/proxy.conf`

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://front-end:9000;
    }

    location /api {
        proxy_pass http://back-end:5252;
    }
}
```

### Explication directive par directive

**`listen 80;`**
Le proxy écoute sur le port `80`, qui est le port HTTP standard. C'est ce qui permet aux clients d'accéder au site simplement via `http://localhost` (sans avoir à préciser de port).

**`location / { proxy_pass http://front-end:9000; }`**
Toute requête qui commence par `/` est **transférée** au service `front-end` sur son port interne `9000`. C'est ainsi que le navigateur reçoit la page HTML, le CSS, le JS, les images, etc.

**`location /api { proxy_pass http://back-end:5252; }`**
Toute requête qui commence par `/api` est transférée au service `back-end` sur le port `5252`. Par exemple, `GET /api/hello` arrive au proxy puis est relayée à Flask.

> **Ordre d'évaluation** : Nginx choisit le bloc `location` le plus **spécifique** qui matche. Une requête `/api/hello` matche les deux `location`, mais `/api` étant plus précis que `/`, c'est lui qui gagne. Le routage fonctionne donc correctement.

### La magie du DNS Docker

Les URLs `http://front-end:9000` et `http://back-end:5252` peuvent surprendre : `front-end` et `back-end` ne sont pas des noms d'hôtes classiques. Ce sont les **noms des services** définis dans `docker-compose.yml`.

Docker Compose crée un **réseau privé** dans lequel chaque conteneur peut résoudre les autres par leur nom de service grâce à un **DNS interne**. Aucune configuration IP manuelle nécessaire : Compose gère tout, et les adresses restent valides même si Docker leur attribue des IPs différentes au redémarrage.

C'est exactement la « magie » mentionnée dans l'énoncé.

## Le `docker-compose.yml` mis à jour

```yaml
services:
  proxy:
    build:
      context: ./proxy
      dockerfile: Dockerfile
    image: softy-pinko-proxy:task5
    ports:
      - "80:80"
    depends_on:
      - back-end
      - front-end

  back-end:
    build:
      context: ./back-end
      dockerfile: Dockerfile
    image: softy-pinko-back-end:task5

  front-end:
    build:
      context: ./front-end
      dockerfile: Dockerfile
    image: softy-pinko-front-end:task5
```

### Les changements importants par rapport à la Task 4

**Ajout du service `proxy`** avec son `build`, son `image`, son mapping `"80:80"` et la directive `depends_on`.

**`depends_on: [back-end, front-end]`**
Indique à Compose de **démarrer** `back-end` et `front-end` **avant** `proxy`. Sans cela, le proxy pourrait démarrer en premier et essayer de joindre des services qui n'existent pas encore. Notez que `depends_on` ne garantit que l'ordre de démarrage, pas que les services soient **prêts** à répondre.

**Suppression des `ports` de `back-end` et `front-end`**
C'est le changement le plus important. Dans la Task 4 :

```yaml
back-end:
  ports:
    - "5252:5252"       # accessible depuis l'extérieur
front-end:
  ports:
    - "9000:9000"       # accessible depuis l'extérieur
```

Dans la Task 5, ces mappings disparaissent. Conséquence :

- Les ports `5252` et `9000` **n'existent plus** sur la machine hôte
- Ces services ne sont **plus joignables depuis le navigateur** (essayez `http://localhost:5252` → erreur de connexion)
- Ils restent **joignables entre conteneurs** via le réseau interne de Compose (c'est ainsi que le proxy peut toujours les atteindre)

C'est exactement ce qu'on veut : seul le proxy est exposé au monde extérieur. Tout passe par lui.

## Le JavaScript mis à jour

L'AJAX du front-end passait par `http://localhost:5252/api/hello`. Désormais, comme tout passe par le proxy, on utilise une **URL relative** :

```html
<script>
    $(function() {
        $.ajax({
            type: "GET",
            url: "/api/hello",
            success: function (data) {
                console.log(data);
                $('#dynamic-content').text(data);
            }
        });
    });
</script>
```

### Pourquoi ce changement ?

- Une URL **absolue** comme `http://localhost:5252/api/hello` cible explicitement un serveur sur un port précis. Or ce port n'est plus exposé.
- Une URL **relative** comme `/api/hello` est résolue par le navigateur en utilisant l'**origine de la page courante**. Si la page est servie depuis `http://localhost`, la requête part vers `http://localhost/api/hello`, donc vers le **proxy** (port 80).
- Le proxy voit l'URL commencer par `/api` et la transfère à `back-end:5252`.

C'est plus propre : le code JS n'a plus besoin de connaître les détails internes de l'infrastructure. Si demain on change le port du back-end, **rien à modifier côté front-end**.

### Bonus : plus besoin de CORS ?

Côté navigateur, le front-end et l'API ont maintenant la **même origine** (`http://localhost`, port 80). Le mécanisme CORS n'est techniquement plus nécessaire. On peut toutefois laisser `flask-cors` actif : il devient inoffensif et permettrait de réutiliser l'API depuis un autre domaine en cas de besoin.

## Construction et lancement

Depuis le dossier `task5/` :

```bash
docker-compose build
docker-compose up
```

Compose construit les **trois** images (back-end, front-end, proxy) puis les lance en parallèle. Trois conteneurs apparaissent dans les logs :

```
 ⠿ Container task5-back-end-1   Created
 ⠿ Container task5-front-end-1  Created
 ⠿ Container task5-proxy-1      Created
Attaching to task5-back-end-1, task5-front-end-1, task5-proxy-1
```

Les logs des trois services s'affichent en parallèle, préfixés par leur nom.

## Tester l'application

Ouvrir [http://localhost](http://localhost) (sans préciser de port → port 80 par défaut).

Le site doit s'afficher avec le titre dynamique `Hello, World!` en haut. Cette fois, tout transite par le proxy :

1. Le navigateur demande `/` → proxy → front-end → Nginx sert `index.html`
2. Le navigateur charge les assets (`/assets/css/...`, `/assets/js/...`) → proxy → front-end
3. Le JavaScript exécute l'AJAX vers `/api/hello` → proxy → back-end → Flask renvoie `Hello, World!`
4. Le texte s'affiche dynamiquement

Dans les logs Compose, on voit la trace :

```
task5-back-end-1   | 172.19.0.4 - - [15/Jun/2023 19:28:48] "GET /api/hello HTTP/1.0" 200 -
```

L'IP `172.19.0.4` est celle du conteneur **proxy** sur le réseau interne. Flask ne voit jamais l'IP du navigateur : c'est le proxy qui lui parle.

## Diagramme de l'architecture

```
                     ┌────────────────────────────────┐
                     │     Réseau Docker interne      │
   Navigateur        │                                │
   localhost:80      │   ┌─────────┐  /api → ┌──────┐ │
   ────────────────► │   │  proxy  │ ───────►│ back │ │
                     │   │ (Nginx) │         │ (Flask)│
                     │   │ port 80 │  / →    │ 5252 │ │
                     │   └─────────┘ ┐       └──────┘ │
                     │               ▼               │
                     │           ┌──────┐            │
                     │           │front │            │
                     │           │(Nginx)│           │
                     │           │ 9000 │            │
                     │           └──────┘            │
                     └────────────────────────────────┘
```

Seul le proxy est exposé sur le port `80` de la machine hôte. Tout le reste est invisible depuis l'extérieur.

## Concepts clés à retenir

- **Reverse proxy** : serveur placé devant d'autres serveurs, qui reçoit les requêtes clients et les redistribue. C'est la base de toute architecture web sérieuse (load balancing, terminaison TLS, cache, sécurité).
- **Point d'entrée unique** : un seul service exposé publiquement. Les autres restent privés sur le réseau interne. Ça simplifie le déploiement et améliore la sécurité.
- **DNS interne de Docker Compose** : chaque service est joignable par son nom (`back-end`, `front-end`) sur le réseau Compose, sans configuration manuelle.
- **URLs relatives vs absolues** : utiliser des URLs relatives dans le code client le rend indépendant de la topologie de l'infrastructure. C'est presque toujours ce qu'on veut.
- **`location` blocks de Nginx** : c'est avec `location` + `proxy_pass` qu'on construit le routage. Le bloc le plus spécifique gagne.
- **`depends_on`** : ordonne le démarrage des services, mais **ne garantit pas qu'ils soient prêts**. Pour ça, il faudrait des health checks.
- **Disparition (volontaire) du CORS** : avec un reverse proxy, front et back partagent la même origine côté navigateur, donc le CORS devient inutile.