# Softy Pinko Docker - Task 2 : Front-end avec Nginx

## Objectif

Réorganiser le projet pour séparer le **back-end** (API Flask) et le **front-end** (site statique). Le front-end est servi par **Nginx**, un serveur web rapide et léger, à partir d'une seconde image Docker dédiée.

## Énoncé

Faire une copie du dossier `task1` et la renommer `task2`, puis réorganiser :

- Créer un dossier `back-end` dans `task2` et y déplacer **tous les fichiers existants** (`api.py` et `Dockerfile`)
- Créer un dossier `front-end` dans `task2`
- Cloner le repo `https://github.com/atlas-school/softy-pinko-front-end` à l'intérieur du dossier `front-end`
- Créer un nouveau `Dockerfile` dans `task2/front-end` basé sur la **dernière version de Nginx** (et non Ubuntu)
- Copier les fichiers du front-end vers `/var/www/html/softy-pinko-front-end` dans l'image
- Créer un fichier de configuration Nginx nommé `softy-pinko-front-end.conf` et le copier dans l'image vers `/etc/nginx/conf.d/default.conf`
- Le serveur doit écouter sur le port `9000`
- Lors du `docker run`, rediriger le port `9000` du conteneur vers le port `9000` de l'hôte

## Structure du projet

```
softy-pinko-docker/
└── task2/
    ├── back-end/
    │   ├── api.py
    │   └── Dockerfile
    └── front-end/
        ├── softy-pinko-front-end/      # cloné depuis GitHub
        │   ├── assets/
        │   └── index.html
        ├── Dockerfile
        └── softy-pinko-front-end.conf
```

Cette séparation reflète une **architecture client-serveur** : le back-end (API) et le front-end (site web) sont deux services indépendants, chacun avec son propre Dockerfile et sa propre image.

## Le back-end (rappel de la Task 1)

Le dossier `back-end` reprend exactement le contenu de la Task 1, déplacé tel quel.

**`back-end/api.py`** : serveur Flask exposant `/api/hello` sur le port `5252`.

**`back-end/Dockerfile`** : image Ubuntu + Python3 + Flask, identique à la Task 1.

## Le front-end : nouveau Dockerfile

```dockerfile
# Utiliser la dernière version de Nginx
FROM nginx:latest

# Copier tous les fichiers du front-end vers le dossier web de Nginx
COPY ./softy-pinko-front-end /var/www/html/softy-pinko-front-end

# Copier la configuration Nginx personnalisée
COPY ./softy-pinko-front-end.conf /etc/nginx/conf.d/default.conf

# Exposer le port 9000 (informatif)
EXPOSE 9000

# Nginx démarre automatiquement avec l'image, pas besoin de CMD
```

### Explication ligne par ligne

**`FROM nginx:latest`**
On part de l'image officielle Nginx (et non Ubuntu comme dans les tâches précédentes). Cette image contient déjà Nginx installé et configuré pour démarrer automatiquement. Cela évite d'avoir à installer et configurer Nginx manuellement.

**`COPY ./softy-pinko-front-end /var/www/html/softy-pinko-front-end`**
Copie tout le contenu du dossier local `softy-pinko-front-end` (HTML, CSS, JS, images, etc.) dans le dossier `/var/www/html/softy-pinko-front-end` à l'intérieur de l'image. C'est l'emplacement standard pour servir des fichiers web sous Linux.

**`COPY ./softy-pinko-front-end.conf /etc/nginx/conf.d/default.conf`**
Remplace le fichier de configuration par défaut de Nginx par notre configuration personnalisée. Nginx charge automatiquement tous les fichiers `.conf` présents dans `/etc/nginx/conf.d/`.

**`EXPOSE 9000`**
Documente que le conteneur écoutera sur le port `9000`. Comme pour la Task 1, c'est purement informatif ; la publication réelle du port se fait via `-p` au moment du `docker run`.

**Pas de `CMD`**
Contrairement aux images Ubuntu, l'image officielle Nginx contient déjà un `CMD` (et un `ENTRYPOINT`) qui démarre Nginx en avant-plan. On hérite donc de ce comportement sans avoir à le redéfinir.

## La configuration Nginx : `softy-pinko-front-end.conf`

```nginx
server {
    listen 9000;
    server_name localhost;
    location / {
        root /var/www/html/softy-pinko-front-end;
        index index.html;
    }
}
```

### Explication directive par directive

**`server { ... }`**
Définit un **bloc serveur** (l'équivalent d'un *virtual host*). Toute la configuration d'un site se place à l'intérieur de ce bloc.

**`listen 9000;`**
Indique à Nginx d'écouter sur le port `9000`. Lorsqu'une requête HTTP arrive sur ce port, ce bloc serveur sera utilisé pour la traiter.

**`server_name localhost;`**
Définit le nom du serveur. Nginx utilise cette valeur pour décider quel bloc serveur traite la requête en cas de plusieurs sites hébergés. En local, `localhost` suffit.

**`location / { ... }`**
Définit comment traiter les requêtes commençant par `/` (donc toutes les requêtes du site).

**`root /var/www/html/softy-pinko-front-end;`**
Indique à Nginx où trouver les fichiers à servir. Le chemin doit correspondre exactement à l'emplacement utilisé dans le `COPY` du Dockerfile.

**`index index.html;`**
Précise le fichier par défaut à servir lorsqu'un client demande un répertoire (par exemple `http://localhost:9000/`). Nginx renverra alors le fichier `index.html`.

## Construction de l'image front-end

Depuis le dossier `task2/` :

```bash
docker build -f ./front-end/Dockerfile -t softy-pinko-front-end:task2 ./front-end
```

### Détail des options

- `-f ./front-end/Dockerfile` : chemin vers le Dockerfile à utiliser
- `-t softy-pinko-front-end:task2` : tag de l'image (nom:version)
- `./front-end` : **contexte de build**. Tous les fichiers que le Dockerfile peut copier doivent être à l'intérieur de ce dossier. C'est pour cela que les chemins du Dockerfile sont relatifs à `./front-end`.

### Résultat de la construction

```
[+] Building 0.6s (8/8) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:latest
 => [internal] load build context
 => [1/3] FROM docker.io/library/nginx:latest
 => CACHED [2/3] COPY ./softy-pinko-front-end /var/www/html/softy-pinko-front-end
 => CACHED [3/3] COPY ./softy-pinko-front-end.conf /etc/nginx/conf.d/default.conf
 => exporting to image
 => => naming to docker.io/library/softy-pinko-front-end:task2
```

## Exécution du conteneur front-end

```bash
docker run -p 9000:9000 -it --rm --name softy-pinko-front-end-task2 softy-pinko-front-end:task2
```

- `-p 9000:9000` : redirection du port `9000` du conteneur vers le port `9000` de la machine hôte (sans cela, le site serait inaccessible depuis le navigateur)
- `-it` : mode interactif avec TTY pour voir les logs de Nginx
- `--rm` : suppression automatique du conteneur à l'arrêt
- `--name softy-pinko-front-end-task2` : nom du conteneur
- `softy-pinko-front-end:task2` : image à exécuter

### Logs attendus

Au démarrage, Nginx affiche notamment :

```
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/06/12 17:00:32 [notice] 1#1: using the "epoll" event method
2023/06/12 17:00:32 [notice] 1#1: nginx/1.25.0
2023/06/12 17:00:32 [notice] 1#1: start worker processes
```

Cela signifie que Nginx est lancé, qu'il a chargé notre configuration, et qu'il attend des requêtes sur le port `9000`.

## Tester le front-end

Ouvrir un navigateur et aller à l'adresse :

[http://localhost:9000](http://localhost:9000)

Le site `softy-pinko-front-end` doit s'afficher (page HTML statique avec ses styles et images).

## Concepts clés à retenir

- **Séparation back-end / front-end** : chaque service est dans son propre dossier, avec son propre Dockerfile et sa propre image. C'est une pratique fondamentale en architecture de microservices.
- **Nginx** est un serveur web qui sert des fichiers statiques de manière très efficace. L'image officielle Nginx est prête à l'emploi : il suffit de copier les fichiers du site et un fichier de configuration.
- **Image officielle vs image custom** : utiliser `nginx:latest` plutôt qu'`ubuntu:24.04` + installation manuelle simplifie énormément le Dockerfile (3 lignes au lieu de 7).
- **`/etc/nginx/conf.d/default.conf`** est le fichier de configuration qu'Nginx charge automatiquement. En l'écrasant via `COPY`, on personnalise le comportement du serveur.
- **Contexte de build** : le dernier argument de `docker build` (`./front-end` ici) délimite les fichiers accessibles aux instructions `COPY`. Tout ce qui se trouve hors de ce dossier est invisible pour le build.
- **Pas besoin de `CMD`** quand l'image de base définit déjà le bon comportement de démarrage (cas de Nginx).