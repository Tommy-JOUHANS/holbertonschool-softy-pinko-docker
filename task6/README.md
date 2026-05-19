# Softy Pinko Docker - Task 6 : Scaling Horizontal

## Objectif

**Multiplier dynamiquement** le nombre d'instances du back-end pour absorber davantage de trafic. Nginx, placé en proxy, distribue automatiquement les requêtes entre les différentes instances grâce à l'algorithme **Round-Robin** (à tour de rôle). On obtient ainsi un mini **load balancer** sans modifier une seule ligne de configuration.

## Pourquoi le scaling horizontal ?

Quand une application reçoit beaucoup de trafic, deux stratégies existent :

- **Scaling vertical** : augmenter la puissance d'une seule machine (plus de CPU, plus de RAM). Limité physiquement, coûteux, ne protège pas contre les pannes.
- **Scaling horizontal** : ajouter **plus de machines/conteneurs** qui font le même travail en parallèle. C'est ce qu'on fait ici.

Le scaling horizontal apporte :

- **Plus de débit** : N serveurs gèrent N fois plus de requêtes
- **Tolérance aux pannes** : si une instance tombe, les autres prennent le relais
- **Élasticité** : on monte ou descend le nombre d'instances en fonction du trafic
- **Pas de modification du code** : c'est entièrement une question d'orchestration

C'est le mode de fonctionnement standard de toutes les grandes plateformes web.

## Énoncé

Faire une copie du dossier `task5` et la renommer `task6`. Puis :

- Lancer Docker Compose en demandant **2 instances** (ou plus) du service `back-end`
- Le proxy Nginx doit répartir les requêtes entre ces instances avec l'algorithme **Round-Robin**
- Créer dans `task6/` un fichier nommé `2-api-servers.txt` contenant **exactement** la commande utilisée pour lancer 2 serveurs d'API
- Le nom du service back-end doit rester `back-end` (le correcteur s'y attend)
- Le fichier `2-api-servers.txt` doit se terminer par une nouvelle ligne

## Structure du projet

```
softy-pinko-docker/
└── task6/
    ├── 2-api-servers.txt          # nouveau (contient la commande de scaling)
    ├── docker-compose.yml
    ├── back-end/
    │   ├── api.py
    │   ├── requirements.txt
    │   └── Dockerfile
    ├── front-end/
    │   ├── softy-pinko-front-end/
    │   ├── Dockerfile
    │   └── softy-pinko-front-end.conf
    └── proxy/
        ├── Dockerfile
        └── proxy.conf
```

Le code et les Dockerfiles sont **identiques** à la Task 5. La nouveauté ne tient qu'à la commande de lancement.

## Le fichier `2-api-servers.txt`

```
docker-compose up --scale back-end=2
```

(suivi d'une nouvelle ligne).

### Explication de la commande

**`docker-compose up`**
Lance tous les services définis dans `docker-compose.yml`, comme dans les tâches précédentes.

**`--scale back-end=2`**
Demande à Compose de lancer **2 instances** du service `back-end` au lieu d'une seule. Les autres services (`front-end`, `proxy`) restent à une seule instance.

Compose nomme automatiquement les conteneurs :

- `task6-back-end-1`
- `task6-back-end-2`

Pour 5 instances, on écrirait `--scale back-end=5` et on obtiendrait `task6-back-end-1` à `task6-back-end-5`.

> Important : **aucune modification du `docker-compose.yml`** n'est nécessaire. Le scaling se fait à la **ligne de commande**, ce qui rend très facile l'ajustement à la volée.

## Comment Nginx fait-il le load balancing ?

C'est le point le plus subtil de cette tâche. La configuration du proxy n'a **pas changé** par rapport à la Task 5 :

```nginx
location /api {
    proxy_pass http://back-end:5252;
}
```

Pourtant, le trafic est bien réparti entre les deux instances. Pourquoi ?

### La magie du DNS Docker (suite)

Quand on a une seule instance du service `back-end`, le DNS interne de Docker résout `back-end` vers **une IP unique**. Quand on en a deux ou plus, il résout `back-end` vers **plusieurs IPs**.

À chaque requête sortante vers `back-end`, Nginx redemande au DNS de Docker quelle IP utiliser. Le résolveur DNS de Docker renvoie les IPs **à tour de rôle** : c'est l'algorithme **Round-Robin DNS**.

Résultat : la première requête part vers `back-end-1`, la deuxième vers `back-end-2`, la troisième vers `back-end-1`, etc. Aucune configuration explicite de load balancing n'est nécessaire dans `proxy.conf` : tout est géré côté DNS par Docker.

### Algorithme Round-Robin

Round-Robin (« tourniquet ») est l'algorithme de répartition le plus simple : on distribue les requêtes **séquentiellement** entre les serveurs disponibles.

| Requête | Serveur cible |
|---------|---------------|
| 1 | back-end-1 |
| 2 | back-end-2 |
| 3 | back-end-1 |
| 4 | back-end-2 |
| 5 | back-end-1 |
| ... | ... |

Avantages : simple, équitable, sans état. Limites : ne tient pas compte de la charge réelle de chaque serveur ni de leurs temps de réponse. Pour des cas plus avancés, Nginx propose `least_conn`, `ip_hash`, etc.

## Lancement et vérification

Depuis le dossier `task6/` :

```bash
docker-compose up --scale back-end=2
```

### Sortie attendue au démarrage

```
[+] Running 4/0
 ⠿ Container task6-back-end-2   Created
 ⠿ Container task6-back-end-1   Created
 ⠿ Container task6-front-end-1  Created
 ⠿ Container task6-proxy-1      Created
Attaching to task6-back-end-1, task6-back-end-2, task6-front-end-1, task6-proxy-1
task6-back-end-1   |  * Running on http://172.19.0.2:5252
task6-back-end-2   |  * Running on http://172.19.0.4:5252
```

On voit bien **deux instances Flask** démarrer, chacune avec sa propre IP interne (`172.19.0.2` et `172.19.0.4`). Les deux écoutent sur le port `5252` à l'intérieur de leur conteneur, sans conflit puisqu'elles sont sur des réseaux isolés.

### Vérification du load balancing

En ouvrant [http://localhost](http://localhost) puis en rechargeant **plusieurs fois** la page, on voit dans les logs :

```
task6-back-end-1   | 172.20.0.5 - - "GET /api/hello HTTP/1.0" 200 -
task6-back-end-2   | 172.20.0.5 - - "GET /api/hello HTTP/1.0" 200 -
task6-back-end-1   | 172.20.0.5 - - "GET /api/hello HTTP/1.0" 200 -
task6-back-end-2   | 172.20.0.5 - - "GET /api/hello HTTP/1.0" 200 -
```

L'**alternance parfaite** entre `back-end-1` et `back-end-2` est la preuve que le Round-Robin fonctionne. L'IP `172.20.0.5` est celle du proxy, qui sert d'unique client aux deux back-ends.

### Avec 5 instances

```bash
docker-compose up --scale back-end=5
```

```
task6-back-end-2   | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-5   | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-1   | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-4   | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-3   | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-2   | "GET /api/hello HTTP/1.0" 200 -
```

Le cycle passe maintenant par les **cinq** instances avant de recommencer. Aucun changement de code ni de configuration : juste une commande.

## Pourquoi ça marche : le rôle de chaque pièce

```
       Navigateur
            │
            ▼
    ┌──────────────┐
    │    proxy     │  reçoit la requête sur /api/hello
    │  (Nginx :80) │  redirige vers http://back-end:5252
    └──────┬───────┘
           │
           ▼
   DNS interne de Docker
   résout "back-end" en Round-Robin
           │
    ┌──────┴──────┐
    ▼             ▼
┌──────────┐  ┌──────────┐
│back-end-1│  │back-end-2│
│  Flask   │  │  Flask   │
│  :5252   │  │  :5252   │
└──────────┘  └──────────┘
```

- Le **proxy** ne change pas : il fait toujours `proxy_pass http://back-end:5252`.
- Le **DNS de Docker** est la pièce magique qui répartit les requêtes en renvoyant alternativement les IPs des conteneurs back-end.
- Les **back-ends** ne savent pas qu'ils font partie d'un cluster ; chaque instance répond comme si elle était seule.

C'est pour cela qu'on parle d'**architecture sans état** (stateless) : chaque requête doit pouvoir être traitée par n'importe quelle instance sans qu'elles aient à se coordonner.

## Limites et améliorations possibles

- **Pas d'état partagé** : si l'API stockait des sessions en mémoire, deux requêtes du même utilisateur pourraient tomber sur des instances différentes. Pour résoudre cela : sessions externalisées (Redis, base de données) ou `ip_hash` côté Nginx.
- **Pas de health check** : si une instance plante, le DNS continuerait à l'inclure dans la rotation pendant un temps. Une vraie production utiliserait Kubernetes ou Docker Swarm pour gérer cela.
- **Pas de scaling automatique** : le nombre d'instances est fixé à la main. Les orchestrateurs modernes (Kubernetes HPA, ECS Auto Scaling) ajustent dynamiquement selon la charge.

Mais pour comprendre les concepts, ce setup est parfait.

## Concepts clés à retenir

- **Scaling horizontal vs vertical** : ajouter des instances (horizontal) est plus flexible et plus résilient que renforcer une seule machine (vertical).
- **Load balancing** : la répartition du trafic entre plusieurs serveurs. Le **Round-Robin** est l'algorithme le plus simple et le plus courant.
- **`--scale service=N`** : option de `docker-compose up` qui multiplie un service à la volée, sans toucher au YAML.
- **DNS Round-Robin de Docker** : quand un nom de service correspond à plusieurs conteneurs, Docker renvoie les IPs **à tour de rôle**. C'est ce qui crée le load balancing implicite.
- **Application stateless** : prérequis du scaling horizontal. Chaque requête doit pouvoir être traitée par n'importe quelle instance, sans dépendance à un état local.
- **Cohérence avec la Task 5** : le client n'a toujours qu'une seule adresse à connaître (`http://localhost`). Toute la complexité de la répartition est cachée derrière le proxy.

## Repo

- **GitHub repository** : `holbertonschool-softy-pinko-docker`
- **Directory** : `task6`