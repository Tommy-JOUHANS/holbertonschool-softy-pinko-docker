# Softy Pinko Docker - Task 0 : Create Your First Docker Image

## Objectif

Créer une première image Docker basée sur Ubuntu, qui met à jour les paquets du système et affiche `Hello, World!` lorsqu'elle est exécutée dans un conteneur.

## Énoncé

Pour créer une image Docker, il faut utiliser un **Dockerfile**. Le Dockerfile doit :

- Être basé sur la dernière version d'**Ubuntu**
- Mettre à jour APT avec `apt-get update`
- Mettre à niveau les logiciels installés avec `apt-get upgrade -y`
- Une fois construite, l'image doit pouvoir être exécutée dans un conteneur et afficher `Hello, World!` dans le terminal

## Structure du projet

```
softy-pinko-docker/
└── task0/
    └── Dockerfile
```

## Le Dockerfile

```dockerfile
FROM ubuntu:24.04
RUN apt-get update
RUN apt-get upgrade -y
CMD ["echo", "Hello, World!"]
```

### Explication ligne par ligne

**`FROM ubuntu:24.04`**
Cette instruction définit l'image de base utilisée pour construire notre image. Ici, on part de la version 24.04 d'Ubuntu (la dernière LTS disponible). Toutes les instructions suivantes seront appliquées sur cette base.

**`RUN apt-get update`**
L'instruction `RUN` exécute une commande pendant la construction de l'image. `apt-get update` met à jour la liste des paquets disponibles dans les dépôts APT. C'est une étape indispensable avant d'installer ou mettre à jour des logiciels.

**`RUN apt-get upgrade -y`**
Cette commande met à niveau tous les paquets déjà installés sur le système vers leur dernière version. L'option `-y` répond automatiquement « oui » à toutes les questions interactives, ce qui est nécessaire car aucun utilisateur ne peut interagir pendant la construction de l'image.

**`CMD ["echo", "Hello, World!"]`**
L'instruction `CMD` définit la commande qui sera exécutée par défaut lorsqu'un conteneur sera lancé à partir de cette image. Ici, on utilise la forme JSON (exec form) qui est la forme recommandée. Elle exécute la commande `echo Hello, World!`.

> Différence importante : `RUN` s'exécute pendant la **construction** (build) de l'image, alors que `CMD` s'exécute lorsque le conteneur **démarre**.

## Construction de l'image

Depuis le dossier `task0/`, on construit l'image avec la commande :

```bash
docker build -f ./Dockerfile -t softy-pinko:task0 .
```

### Explication des options

- `docker build` : commande de construction d'une image Docker
- `-f ./Dockerfile` : spécifie le chemin du Dockerfile à utiliser
- `-t softy-pinko:task0` : applique un tag (nom:version) à l'image (`softy-pinko` est le nom du repository, `task0` le tag)
- `.` : indique le contexte de construction (le dossier courant), c'est-à-dire l'ensemble des fichiers que Docker peut utiliser pendant la construction

### Résultat de la construction

```
[+] Building 36.9s (8/8) FINISHED                              docker:default
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 131B
 => [internal] load metadata for docker.io/library/ubuntu:24.04
 => [auth] library/ubuntu:pull token for registry-1.docker.io
 => [internal] load .dockerignore
 => CACHED [1/3] FROM docker.io/library/ubuntu:24.04
 => [2/3] RUN apt-get update                                              21.8s
 => [3/3] RUN apt-get upgrade -y                                           8.7s
 => exporting to image
 => => exporting layers
 => => writing image sha256:b36ad2dc4d80511e59d0c147ab1c4ffe6596bb81f20748ca49ccbd74f4416720
 => => naming to docker.io/library/softy-pinko:task0
```

Chaque étape du Dockerfile correspond à une **couche** (layer) de l'image. Docker met en cache ces couches : si on reconstruit l'image sans avoir modifié les instructions, les étapes seront marquées `CACHED` et la construction sera quasi instantanée.

## Exécution du conteneur

Pour lancer un conteneur basé sur l'image construite :

```bash
docker run -it --rm --name softy-pinko-task0 softy-pinko:task0
```

### Explication des options

- `docker run` : crée et démarre un nouveau conteneur
- `-i` : mode interactif (garde STDIN ouvert)
- `-t` : alloue un pseudo-terminal (TTY)
- `--rm` : supprime automatiquement le conteneur après son arrêt (pratique pour ne pas accumuler des conteneurs inutilisés)
- `--name softy-pinko-task0` : donne un nom au conteneur
- `softy-pinko:task0` : nom de l'image à utiliser

### Résultat de l'exécution

```
Hello, World!
```

La commande définie dans `CMD` s'exécute, affiche le message, puis le conteneur s'arrête immédiatement (et est supprimé grâce à `--rm`).

## Récapitulatif du processus

1. **Écrire le Dockerfile** avec les instructions nécessaires (base, mises à jour, commande par défaut).
2. **Construire l'image** avec `docker build` en lui donnant un tag.
3. **Exécuter un conteneur** à partir de l'image avec `docker run`.
4. **Vérifier la sortie** : le terminal doit afficher `Hello, World!`.

## Concepts clés à retenir

- Une **image** Docker est un modèle immuable contenant tout ce qu'il faut pour exécuter une application.
- Un **conteneur** est une instance en cours d'exécution d'une image.
- Le **Dockerfile** est la recette qui décrit comment construire l'image.
- Les **couches** (layers) permettent à Docker de mettre en cache les étapes et d'accélérer les reconstructions.
- `RUN` s'exécute au moment du *build*, `CMD` s'exécute au moment du *run*.