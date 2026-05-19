# Softy Pinko Docker - Task 1 : Back-end Flask

## Objectif

Faire évoluer l'image Docker de la Task 0 pour qu'elle exécute un **serveur back-end Flask** écrit en Python. Le serveur expose un endpoint `/api/hello` qui retourne `Hello, World!` lorsqu'on l'appelle.

## Énoncé

Faire une copie du dossier `task0` et la renommer `task1`. Modifier le `Dockerfile` pour :

- Installer `python3`, `python3-pip` et `flask`
- Utiliser le flag `-y` pour `apt-get` afin d'éviter les questions interactives
- Installer **Flask avec `pip3`**, pas avec `apt-get`
- Si l'erreur `This environment is externally managed` apparaît, ajouter la ligne `RUN rm /usr/lib/python*/EXTERNALLY-MANAGED` avant l'appel à `pip`
- Définir `/app` comme répertoire de travail (`WORKDIR`)
- Copier le fichier `api.py` dans l'image
- Héberger l'application Flask sur `0.0.0.0` (et non `127.0.0.1`) pour qu'elle soit accessible depuis l'extérieur du conteneur
- Exposer le port `5252`
- Au lancement du conteneur, rediriger (forward) le port `5252` du conteneur vers le port `5252` de la machine hôte

## Structure du projet

```
softy-pinko-docker/
├── task0/
│   └── Dockerfile
└── task1/
    ├── Dockerfile
    └── api.py
```

## Le fichier `api.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route('/api/hello')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5252)
```

### Explication ligne par ligne

**`from flask import Flask`**
Importe la classe `Flask` depuis la librairie Flask. C'est le micro-framework web utilisé pour créer notre API.

**`app = Flask(__name__)`**
Crée une instance de l'application Flask. La variable `__name__` permet à Flask de savoir où chercher les ressources (templates, fichiers statiques, etc.).

**`@app.route('/api/hello')`**
Décorateur qui associe une URL (`/api/hello`) à la fonction qui suit. Lorsqu'un client envoie une requête HTTP `GET` sur cette URL, Flask exécute la fonction décorée.

**`def hello_world(): return 'Hello, World!'`**
Définit la fonction du endpoint. Elle retourne simplement la chaîne de caractères `Hello, World!`, que Flask renverra dans la réponse HTTP.

**`if __name__ == '__main__':`**
Cette condition vérifie que le fichier est exécuté directement (et non importé comme module). C'est le point d'entrée du script.

**`app.run(host='0.0.0.0', port=5252)`**
Démarre le serveur Flask.

- `host='0.0.0.0'` : indique au serveur d'écouter sur **toutes les interfaces réseau** du conteneur. Si on utilisait `127.0.0.1`, le serveur n'écouterait que sur l'interface locale du conteneur, et il serait alors **inaccessible** depuis la machine hôte.
- `port=5252` : port d'écoute du serveur.

## Le Dockerfile

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y python3 python3-pip
RUN rm /usr/lib/python*/EXTERNALLY-MANAGED || true
RUN pip3 install flask

WORKDIR /app
COPY api.py /app

EXPOSE 5252
CMD ["python3", "api.py"]
```

### Explication ligne par ligne

**`FROM ubuntu:24.04`**
Image de base : Ubuntu 24.04 LTS, comme dans la Task 0.

**`RUN apt-get update && apt-get install -y python3 python3-pip`**
Met à jour la liste des paquets APT puis installe Python 3 et pip3 dans la même couche. Combiner les deux commandes avec `&&` est une bonne pratique : cela réduit le nombre de couches et garantit que la liste des paquets est à jour avant l'installation. Le flag `-y` accepte automatiquement les prompts.

**`RUN rm /usr/lib/python*/EXTERNALLY-MANAGED || true`**
Depuis Ubuntu 23.04, Python interdit par défaut l'installation de paquets via `pip` au niveau système (PEP 668). Ce blocage est matérialisé par un fichier `EXTERNALLY-MANAGED`. Le supprimer permet à `pip3` de fonctionner librement. Le `|| true` empêche le build d'échouer si le fichier n'existe pas.

**`RUN pip3 install flask`**
Installe Flask via pip3 (et non via apt-get, comme l'exige l'énoncé). Cela permet d'avoir la dernière version stable de Flask.

**`WORKDIR /app`**
Définit `/app` comme répertoire de travail. Si le dossier n'existe pas, il est créé. Toutes les instructions suivantes (`COPY`, `CMD`, etc.) s'exécuteront dans ce répertoire.

**`COPY api.py /app`**
Copie le fichier `api.py` du contexte de build (la machine hôte) vers le dossier `/app` de l'image.

**`EXPOSE 5252`**
Indique que le conteneur écoutera sur le port `5252`. C'est une instruction **documentaire** : elle ne publie pas réellement le port. La publication se fait au moment du `docker run` avec l'option `-p`.

**`CMD ["python3", "api.py"]`**
Commande exécutée au démarrage du conteneur : lance le serveur Flask en exécutant `api.py` avec Python 3.

## Construction de l'image

Depuis le dossier `task1/` :

```bash
docker build -f ./Dockerfile -t softy-pinko:task1 .
```

Docker exécute chaque instruction du Dockerfile dans l'ordre et crée une couche pour chacune :

```
[+] Building 0.9s (12/12) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/ubuntu:latest
 => [internal] load build context
 => [1/7] FROM docker.io/library/ubuntu:latest
 => CACHED [2/7] RUN apt-get update
 => CACHED [3/7] RUN apt-get upgrade -y
 => CACHED [4/7] RUN apt-get install -y python3 python3-pip
 => CACHED [5/7] RUN pip3 install flask
 => CACHED [6/7] WORKDIR /app
 => CACHED [7/7] COPY ./api.py /app/api.py
 => exporting to image
 => => writing image sha256:58f5eb04ef4a3ac604fcc74adc799c09e09b2697675d9ec552d45c3a9e7d572
 => => naming to docker.io/library/softy-pinko:task1
```

## Exécution du conteneur

```bash
docker run -p 5252:5252 -it --rm --name softy-pinko-task1 softy-pinko:task1
```

### Explication des options

- `docker run` : crée et démarre un conteneur
- `-p 5252:5252` : **redirection de port** (`host:container`). Le port `5252` du conteneur est mappé sur le port `5252` de la machine hôte. Sans cette option, le serveur Flask serait inaccessible depuis l'extérieur du conteneur.
- `-it` : mode interactif avec TTY, utile pour voir les logs et arrêter le serveur avec `CTRL+C`
- `--rm` : supprime le conteneur à son arrêt
- `--name softy-pinko-task1` : nom du conteneur
- `softy-pinko:task1` : image à exécuter

### Résultat attendu

```
 * Serving Flask app 'api'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5252
 * Running on http://172.17.0.2:5252
Press CTRL+C to quit
```

## Tester le serveur

Une fois le conteneur lancé, on peut tester le endpoint depuis la machine hôte avec :

```bash
curl http://localhost:5252/api/hello
```

Réponse attendue :

```
Hello, World!
```

Ou directement dans un navigateur, en allant à l'adresse [http://localhost:5252/api/hello](http://localhost:5252/api/hello).

## Concepts clés à retenir

- **Flask** est un micro-framework Python qui permet de créer des applications web et des APIs très simplement avec des décorateurs (`@app.route`).
- **`0.0.0.0` vs `127.0.0.1`** : `0.0.0.0` écoute sur toutes les interfaces réseau, ce qui est obligatoire dans un conteneur Docker pour que le serveur soit accessible depuis l'hôte. `127.0.0.1` ne serait accessible que depuis l'intérieur du conteneur.
- **`EXPOSE` vs `-p`** : `EXPOSE` dans le Dockerfile **documente** le port utilisé ; `-p host:container` au `docker run` **publie réellement** le port sur la machine hôte.
- **`WORKDIR`** crée et utilise un dossier comme répertoire de travail à l'intérieur de l'image, ce qui évite de répéter le chemin dans chaque instruction.
- **`COPY`** transfère des fichiers du contexte de build vers l'image. Sans cette étape, `api.py` n'existerait pas dans le conteneur.
- **PEP 668 / EXTERNALLY-MANAGED** : depuis Ubuntu 23.04, l'installation de paquets Python via `pip` au niveau système est bloquée par défaut pour éviter de casser le Python du système. Le fichier `EXTERNALLY-MANAGED` doit être supprimé pour autoriser `pip3 install`.