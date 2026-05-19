# Softy Pinko Docker - Task 3 : Connecter le front-end et le back-end

## Objectif

Faire **communiquer** le front-end Nginx et le back-end Flask. Le navigateur affiche la page statique servie par Nginx, puis exécute une requête AJAX vers l'API Flask pour récupérer dynamiquement le texte `Hello, World!` et l'injecter dans la page.

C'est la première vraie architecture **client-serveur** du projet : deux images Docker, deux conteneurs, deux ports, qui dialoguent ensemble.

## Énoncé

Faire une copie du dossier `task2` et la renommer `task3`. Puis :

- Ajouter dans `index.html` un `<h1 id="dynamic-content"></h1>` juste **avant** le titre « We provide the best strategy to grow up your business »
- Ajouter un script jQuery AJAX près du `</body>` qui appelle `http://localhost:5252/api/hello` et injecte la réponse dans `#dynamic-content`
- Dans le `Dockerfile` du back-end, installer `flask-cors` avec `pip3` après `flask`
- Mettre à jour `api.py` pour importer `flask_cors` et activer `CORS(app)`
- Lancer **deux conteneurs** simultanément (un par terminal) : le back-end sur le port `5252` et le front-end sur le port `9000`

## Structure du projet

```
softy-pinko-docker/
└── task3/
    ├── back-end/
    │   ├── api.py
    │   ├── requirements.txt
    │   └── Dockerfile
    └── front-end/
        ├── softy-pinko-front-end/
        │   ├── assets/
        │   └── index.html         # modifié (h1 + script AJAX)
        ├── Dockerfile
        └── softy-pinko-front-end.conf
```

## Le back-end mis à jour

### `back-end/api.py`

```python
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

@app.route('/api/hello')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5252)
```

### Pourquoi `flask_cors` ?

Par défaut, les navigateurs appliquent la **Same-Origin Policy** : du JavaScript chargé depuis une origine (par ex. `http://localhost:9000`) **ne peut pas** appeler une API hébergée sur une autre origine (par ex. `http://localhost:5252`). Cette protection bloque les requêtes inter-domaines pour éviter les attaques de type CSRF/XSS.

Ici, le front-end (port `9000`) et le back-end (port `5252`) sont sur deux origines différentes : le navigateur considère cela comme du **Cross-Origin**.

**`CORS(app)`** active le mécanisme **Cross-Origin Resource Sharing** : Flask ajoute automatiquement le header HTTP `Access-Control-Allow-Origin: *` à toutes ses réponses, ce qui autorise les requêtes provenant de n'importe quelle origine. Sans cela, le navigateur refuserait d'utiliser la réponse de l'API.

### `back-end/requirements.txt`

```
Flask
Flask-CORS
```

Plutôt que d'enchaîner plusieurs `RUN pip3 install ...` dans le Dockerfile, on liste les dépendances dans un fichier `requirements.txt`. C'est une **bonne pratique** Python : le fichier est versionné, lisible, et facile à mettre à jour.

### `back-end/Dockerfile`

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y python3 python3-pip
RUN rm /usr/lib/python*/EXTERNALLY-MANAGED || true

WORKDIR /app

COPY requirements.txt /app
RUN pip install -r /app/requirements.txt

COPY api.py /app

EXPOSE 5252
CMD ["python3", "api.py"]
```

### Explication des nouveautés par rapport à la Task 1

**`COPY requirements.txt /app` puis `RUN pip install -r /app/requirements.txt`**
On copie d'abord le fichier des dépendances, puis on installe toutes les libs Python en une seule commande. C'est plus **maintenable** : ajouter une nouvelle dépendance se fait dans un fichier texte, sans modifier le Dockerfile.

**Ordre des `COPY` (optimisation du cache Docker)**
Le `COPY requirements.txt` est fait **avant** le `COPY api.py`. Ainsi, si `api.py` change mais pas les dépendances, Docker réutilise le cache de l'étape `pip install` (qui est longue) et ne réinstalle pas Flask/Flask-CORS inutilement. C'est un pattern très courant pour accélérer les rebuilds.

## Le front-end mis à jour

### Modifications dans `index.html`

**1. Ajout d'un titre dynamique vide**

Juste avant le titre principal, on insère :

```html
<h1 id="dynamic-content"></h1>
<h1>We provide the best <strong>strategy</strong><br>to grow up your <strong>business</strong></h1>
```

Le `<h1>` avec l'`id="dynamic-content"` est vide au chargement de la page. Il sera **rempli dynamiquement** par JavaScript après réception de la réponse de l'API.

**2. Script AJAX avant `</body>`**

```html
<script>
    // Load dynamic data from the back-end on port 5252
    $(function() {
        $.ajax({
            type: "GET",
            url: "http://localhost:5252/api/hello",
            success: function(data) {
                console.log(data);
                $('#dynamic-content').text(data);
            }
        });
    });
</script>
```

### Explication du script

**`$(function() { ... })`**
C'est le raccourci jQuery pour `$(document).ready(...)`. Le code à l'intérieur ne s'exécute qu'une fois le DOM entièrement chargé, ce qui garantit que l'élément `#dynamic-content` existe bien.

**`$.ajax({...})`**
Lance une requête HTTP asynchrone via jQuery.

- `type: "GET"` : méthode HTTP utilisée
- `url: "http://localhost:5252/api/hello"` : URL complète de l'endpoint Flask. Notez le port `5252` qui correspond à celui du conteneur back-end publié sur la machine hôte.
- `success: function(data) { ... }` : callback exécuté si la requête réussit. `data` contient la réponse du serveur (`Hello, World!`).

**`$('#dynamic-content').text(data)`**
Sélectionne l'élément avec l'`id="dynamic-content"` et remplace son texte par la valeur reçue de l'API.

**`console.log(data)`**
Affiche aussi la valeur dans la console du navigateur, utile pour le debug.

### Flux complet de la requête

1. Le navigateur charge `http://localhost:9000` (servi par Nginx).
2. Le HTML, le CSS, le JS et jQuery se chargent.
3. Le script AJAX s'exécute et envoie une requête `GET http://localhost:5252/api/hello`.
4. Flask reçoit la requête sur le port `5252`, exécute la fonction `hello_world()` et retourne `Hello, World!` avec les bons headers CORS.
5. Le callback `success` injecte `Hello, World!` dans le `<h1 id="dynamic-content">`.
6. L'utilisateur voit le texte apparaître dynamiquement.

## Construction et exécution

Il faut **deux terminaux**, un pour chaque conteneur.

### Terminal 1 : back-end

```bash
docker build -f ./back-end/Dockerfile -t softy-pinko-back-end:task3 ./back-end
docker run -p 5252:5252 -it --rm --name softy-pinko-back-end-task3 softy-pinko-back-end:task3
```

Sortie attendue :

```
 * Serving Flask app 'api'
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5252
 * Running on http://172.17.0.2:5252
Press CTRL+C to quit
```

### Terminal 2 : front-end

```bash
docker build -f ./front-end/Dockerfile -t softy-pinko-front-end:task3 ./front-end
docker run -p 9000:9000 -it --rm --name softy-pinko-front-end-task3 softy-pinko-front-end:task3
```

Sortie attendue :

```
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/06/12 19:10:39 [notice] 1#1: nginx/1.25.0
2023/06/12 19:10:39 [notice] 1#1: start worker processes
```

### Pourquoi deux terminaux ?

L'option `-it` laisse les conteneurs au premier plan pour afficher leurs logs. Si on lance les deux dans le même terminal, on bloque dès le premier. Deux terminaux permettent de **voir les logs des deux serveurs en temps réel**, très utile pour le débogage.

> Alternative : utiliser `-d` (detached) pour lancer les deux dans un seul terminal en arrière-plan, mais on perd la visibilité immédiate sur les logs. C'est précisément ce qu'on automatisera dans la Task 4 avec `docker-compose`.

## Tester la communication

Ouvrir [http://localhost:9000](http://localhost:9000) dans un navigateur.

Sur la page d'accueil, **au-dessus** du titre « We provide the best strategy... », un nouveau titre doit apparaître : `Hello, World!`. Ce texte ne vient pas du HTML statique, mais bien de l'**API Flask** : la communication front-end ↔ back-end fonctionne.

En ouvrant la console du navigateur (`F12` → Console), on voit aussi le log : `Hello, World!`.

Côté terminal back-end, à chaque rechargement de la page on voit une ligne du type :

```
172.17.0.1 - - [19/May/2026 ...] "GET /api/hello HTTP/1.1" 200 -
```

C'est la trace HTTP de la requête AJAX qui a bien atteint Flask.

## Concepts clés à retenir

- **Architecture client-serveur** : le front-end (Nginx) et le back-end (Flask) sont deux services **indépendants** qui communiquent par HTTP. Chacun tourne dans son propre conteneur, sur son propre port.
- **Same-Origin Policy / CORS** : un navigateur bloque par défaut les requêtes JavaScript vers une origine différente. `flask-cors` ajoute le header `Access-Control-Allow-Origin` qui dit explicitement au navigateur : « tu as le droit de lire ma réponse ».
- **AJAX (Asynchronous JavaScript And XML)** : technique pour faire des requêtes HTTP depuis le navigateur **sans recharger la page**. Ici utilisée via jQuery (`$.ajax`), mais on peut aussi utiliser `fetch()` en JS natif.
- **`requirements.txt`** : meilleure pratique pour gérer les dépendances Python. Plus maintenable et plus rapide qu'enchaîner des `pip install` dans le Dockerfile.
- **Ordre des `COPY` et cache Docker** : copier les dépendances (qui changent rarement) **avant** le code applicatif (qui change souvent) permet à Docker de réutiliser le cache de la couche `pip install` lors des rebuilds.
- **Multi-conteneurs** : faire tourner plusieurs services à la fois est la base des applications modernes. Pour la suite, `docker-compose` automatisera ce lancement.