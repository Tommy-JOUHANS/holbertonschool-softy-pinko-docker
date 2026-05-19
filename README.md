# Softy Pinko Docker - Résumé de l'ensemble des tâches

Ce document synthétise le projet Docker complet, en montrant la **progression pédagogique** de la Task 0 à la Task 6. Chaque tâche ajoute une brique supplémentaire pour passer d'un simple « Hello World » à une **architecture web scalable** avec proxy, load balancer et services multiples.

## Vue d'ensemble : la progression

| Tâche | Concept central | Ce qu'on construit |
|-------|-----------------|---------------------|
| **Task 0** | Première image Docker | Ubuntu qui affiche `Hello, World!` |
| **Task 1** | Conteneur applicatif | Serveur Flask qui répond sur `/api/hello` |
| **Task 2** | Multi-services | Ajout d'un front-end Nginx servant un site statique |
| **Task 3** | Communication inter-services | Le front-end appelle l'API via AJAX + CORS |
| **Task 4** | Orchestration | `docker-compose.yml` lance tout en une commande |
| **Task 5** | Reverse proxy | Nginx en façade route `/` et `/api` vers les bons services |
| **Task 6** | Scaling horizontal | N instances du back-end, load balancing Round-Robin |

Chaque tâche **réutilise** ce qui a été fait avant et y ajoute une couche. À la fin, on a une **mini-architecture de production**.

---

## Task 0 : Créer sa première image Docker

**But** : comprendre ce qu'est un Dockerfile, une image, un conteneur.

**Le Dockerfile** :

```dockerfile
FROM ubuntu:24.04
RUN apt-get update
RUN apt-get upgrade -y
CMD ["echo", "Hello, World!"]
```

**Commandes clés** :

```bash
docker build -f ./Dockerfile -t softy-pinko:task0 .
docker run -it --rm --name softy-pinko-task0 softy-pinko:task0
```

**Acquis** :

- Une **image** est un modèle immuable, un **conteneur** est une instance en cours d'exécution
- `RUN` s'exécute au **build**, `CMD` s'exécute au **run**
- Chaque instruction crée une **couche** mise en cache par Docker

---

## Task 1 : Back-end Flask

**But** : faire tourner une vraie application (un serveur web) dans un conteneur.

**Le fichier `api.py`** : un Flask minimal qui expose `/api/hello` retournant `Hello, World!` sur le port `5252`, en écoutant sur `0.0.0.0` pour être joignable depuis l'extérieur du conteneur.

**Le Dockerfile** ajoute Python, pip, Flask, contourne `EXTERNALLY-MANAGED`, copie `api.py` dans `/app` et démarre le serveur.

**Commandes** :

```bash
docker build -f ./Dockerfile -t softy-pinko:task1 .
docker run -p 5252:5252 -it --rm --name softy-pinko-task1 softy-pinko:task1
```

**Acquis** :

- `0.0.0.0` vs `127.0.0.1` : indispensable pour qu'un service dans un conteneur soit joignable depuis l'hôte
- `EXPOSE` est documentaire ; **`-p hôte:conteneur`** publie réellement le port
- `WORKDIR` et `COPY` permettent d'embarquer du code applicatif dans une image

---

## Task 2 : Front-end Nginx

**But** : séparer le code en deux services (back-end / front-end), introduire Nginx pour servir un site statique.

**Réorganisation** :

```
task2/
├── back-end/        (api.py + Dockerfile de la Task 1)
└── front-end/
    ├── softy-pinko-front-end/   (HTML, CSS, JS cloné depuis GitHub)
    ├── Dockerfile
    └── softy-pinko-front-end.conf
```

**Le Dockerfile Nginx** :

```dockerfile
FROM nginx:latest
COPY ./softy-pinko-front-end /var/www/html/softy-pinko-front-end
COPY ./softy-pinko-front-end.conf /etc/nginx/conf.d/default.conf
EXPOSE 9000
```

**La config Nginx** définit un bloc `server` qui écoute sur le port `9000` et sert `index.html` depuis `/var/www/html/softy-pinko-front-end`.

**Acquis** :

- Utiliser une **image officielle** (`nginx:latest`) plutôt que de tout réinstaller : plus court, plus fiable
- Pas besoin de `CMD` quand l'image de base démarre déjà le bon processus
- Le **contexte de build** (`./front-end`) délimite les fichiers accessibles aux `COPY`

---

## Task 3 : Connexion front-end ↔ back-end

**But** : faire dialoguer les deux services. Le navigateur charge la page (Nginx), puis appelle l'API (Flask) en AJAX et injecte la réponse dans la page.

**Modifications côté front-end** :

- Ajout dans `index.html` d'un `<h1 id="dynamic-content"></h1>` vide
- Ajout d'un script jQuery qui fait `GET http://localhost:5252/api/hello` et écrit la réponse dans cet élément

**Modifications côté back-end** :

- Ajout de **`flask-cors`** dans les dépendances pour autoriser les requêtes inter-origines
- `CORS(app)` dans `api.py`
- Passage à un fichier **`requirements.txt`** (bonne pratique Python)

**Acquis** :

- **Same-Origin Policy** : par défaut, un JS ne peut pas appeler une API sur une autre origine. **CORS** lève cette restriction.
- **AJAX** : faire des requêtes HTTP **sans recharger** la page (jQuery `$.ajax` ou `fetch()`)
- **Ordre des `COPY` et cache Docker** : copier les dépendances avant le code applicatif accélère les rebuilds

---

## Task 4 : Docker Compose

**But** : automatiser le lancement de plusieurs conteneurs avec **une seule commande**.

**Le fichier `docker-compose.yml`** :

```yaml
services:
  back-end:
    build:
      context: ./back-end
      dockerfile: Dockerfile
    image: softy-pinko-back-end:task4
    ports:
      - "5252:5252"

  front-end:
    build:
      context: ./front-end
      dockerfile: Dockerfile
    image: softy-pinko-front-end:task4
    ports:
      - "9000:9000"
```

**Commandes** :

```bash
docker-compose build
docker-compose up
```

**Acquis** :

- **Orchestration** : décrire l'état souhaité (YAML déclaratif) plutôt qu'enchaîner des commandes
- Compose crée un **réseau privé** et un **DNS interne** : chaque service est joignable par son nom
- Une seule commande pour tout lancer, peu importe le nombre de services

---

## Task 5 : Serveur Proxy (Reverse Proxy)

**But** : placer un Nginx en façade. Les clients ne parlent plus qu'à lui ; il route les requêtes vers le bon service en interne.

**Nouveau dossier `proxy/`** avec un `Dockerfile` Nginx et `proxy.conf` :

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

**Modifications du `docker-compose.yml`** :

- Ajout du service `proxy` avec mapping `"80:80"` et `depends_on: [back-end, front-end]`
- **Suppression** des mappings de ports pour `back-end` et `front-end` : ils deviennent **privés** au réseau Compose

**Modification du JS** : passage à une URL **relative** `/api/hello` au lieu de `http://localhost:5252/api/hello`.

**Acquis** :

- **Reverse proxy** : un seul point d'entrée public, services internes cachés, sécurité renforcée
- Les services Compose se résolvent **par leur nom** (`front-end`, `back-end`) grâce au DNS interne
- Les URLs **relatives** rendent le code client indépendant de la topologie
- Plus de problème de CORS car tout passe par la même origine (`http://localhost`)

---

## Task 6 : Scaling Horizontal

**But** : lancer plusieurs instances du back-end pour absorber davantage de trafic. Nginx les utilise en **Round-Robin** automatiquement.

**Aucun changement de code ni de configuration**. Juste la commande de lancement :

```bash
docker-compose up --scale back-end=2
```

Et le fichier **`2-api-servers.txt`** qui contient cette commande (suivi d'un newline) pour le checker.

**Comment ça marche** : quand le service `back-end` a plusieurs instances, le DNS interne de Docker renvoie les IPs **à tour de rôle**. Nginx, qui fait toujours `proxy_pass http://back-end:5252`, voit donc une IP différente à chaque résolution → load balancing implicite.

**Vérification dans les logs** :

```
task6-back-end-1 | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-2 | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-1 | "GET /api/hello HTTP/1.0" 200 -
task6-back-end-2 | "GET /api/hello HTTP/1.0" 200 -
```

L'alternance parfaite prouve que le Round-Robin fonctionne.

**Acquis** :

- **Scaling horizontal** : ajouter des instances plutôt que renforcer une seule machine
- **Round-Robin DNS** : algorithme de répartition le plus simple, intégré nativement à Docker
- **Application stateless** : prérequis indispensable pour que n'importe quelle instance puisse traiter n'importe quelle requête

---

## L'architecture finale (Task 6)

```
                                Internet / Navigateur
                                         │
                                         ▼
                                   ┌──────────┐
                                   │  proxy   │   port 80 (public)
                                   │ (Nginx)  │
                                   └────┬─────┘
                            ┌───────────┴───────────┐
                       /    │                       │   /api
                            ▼                       ▼
                      ┌──────────┐         ┌─────────────────┐
                      │ front-end│         │  back-end-1     │
                      │ (Nginx   │         │  (Flask :5252)  │
                      │  :9000)  │         └─────────────────┘
                      └──────────┘         ┌─────────────────┐
                                           │  back-end-2     │
                                           │  (Flask :5252)  │
                                           └─────────────────┘
                                                  ...
                                           ┌─────────────────┐
                                           │  back-end-N     │
                                           └─────────────────┘

                     ←─────── Réseau Docker interne ────────→
                             (DNS, isolation, ports privés)
```

Seul le **port 80 du proxy** est exposé. Tout le reste vit dans le réseau privé créé par Docker Compose.

---

## Compétences acquises au fil du projet

### Docker pur

- Écrire un **Dockerfile** clair et idiomatique
- Choisir une **image de base** appropriée (Ubuntu pour Python, image officielle pour Nginx)
- Gérer le **cache** en ordonnant intelligemment les instructions
- Construire et taguer des images (`docker build -t`)
- Lancer des conteneurs (`docker run`) avec les bonnes options : `-p`, `--rm`, `--name`, `-it`, `-d`

### Docker Compose

- Déclarer des **services** dans un `docker-compose.yml`
- Comprendre le **réseau interne** et le DNS automatique entre services
- Utiliser `depends_on`, `build`, `image`, `ports`
- Utiliser `--scale` pour scaler horizontalement à la volée
- Naviguer dans les logs multi-conteneurs

### Architecture web

- Différencier **front-end statique** (Nginx) et **back-end dynamique** (Flask)
- Comprendre la **Same-Origin Policy** et le rôle de **CORS**
- Mettre en place un **reverse proxy** avec routage par préfixe d'URL
- Faire du **load balancing** par Round-Robin
- Concevoir une application **stateless** prête au scaling

### Bonnes pratiques

- Utiliser un **`requirements.txt`** pour les dépendances Python
- Préférer les **URLs relatives** dans le code client
- Ne pas exposer inutilement les ports internes
- Versionner toute la configuration d'infrastructure (Dockerfile, docker-compose.yml, conf Nginx)

---

## Conclusion

En sept tâches, le projet va d'un simple `echo Hello, World!` à une **architecture web complète, scalable et reproductible**. Toutes les pièces qu'on y manipule (conteneurs, orchestration, proxy, load balancing) sont les **mêmes fondations** que celles utilisées en production dans des entreprises de toutes tailles, depuis les startups jusqu'aux GAFAM. C'est aussi le tremplin naturel vers des outils plus avancés comme **Docker Swarm** ou **Kubernetes**, qui automatisent à plus grande échelle ce qu'on a fait ici à la main.