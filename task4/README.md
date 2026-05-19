# Softy Pinko Docker - Task 4 : Simplifier avec Docker Compose

## Objectif

Lancer **toute l'application en une seule commande** grâce à **Docker Compose**. Au lieu d'ouvrir deux terminaux et d'enchaîner plusieurs `docker build` et `docker run`, on déclare l'ensemble des services dans un fichier `docker-compose.yml`, et Docker s'occupe de tout : construction des images, création des conteneurs, mise en réseau, démarrage simultané.

## Pourquoi Docker Compose ?

Quand une application n'a qu'un seul conteneur, lancer `docker build` puis `docker run` reste gérable. Mais dès qu'il y a plusieurs composants (back-end, front-end, base de données, cache, file de messages, etc.), enchaîner les commandes à la main devient :

- **répétitif** : il faut retaper les bons ports, les bons noms, les bons tags à chaque démarrage
- **fragile** : oublier une option (`-p`, `--name`, `--rm`) casse le déploiement
- **non versionné** : la « configuration » d'exécution n'existe que dans la mémoire du développeur

Docker Compose résout ces problèmes : la configuration est **déclarative** (dans un fichier YAML versionné dans Git), **reproductible** (toute personne qui clone le repo peut lancer l'app), et **scalable** (10 services se lancent aussi simplement qu'un seul).

## Énoncé

Faire une copie du dossier `task3` et la renommer `task4`. Puis :

- Créer un fichier `docker-compose.yml` à la racine de `task4`
- Y déclarer les **deux services** (`back-end` et `front-end`)
- Pour chacun : indiquer le **contexte de build**, le **Dockerfile**, le **nom de l'image** et la **redirection de port**
- Construire les images avec `docker-compose build`
- Lancer l'application avec `docker-compose up`

Mots-clés importants à utiliser : `services`, `build`, `context`, `dockerfile`, `image`, `ports`, `depends_on`.

## Structure du projet

```
softy-pinko-docker/
└── task4/
    ├── docker-compose.yml          # nouveau
    ├── back-end/
    │   ├── api.py
    │   ├── requirements.txt
    │   └── Dockerfile
    └── front-end/
        ├── softy-pinko-front-end/
        ├── Dockerfile
        └── softy-pinko-front-end.conf
```

Les sous-dossiers `back-end` et `front-end` sont **identiques** à ceux de la Task 3. La seule nouveauté est le fichier `docker-compose.yml`.

## Le fichier `docker-compose.yml`

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

### Explication directive par directive

**`services:`**
Liste tous les services (= conteneurs) qui composent l'application. Chaque service correspond à une image Docker qu'on souhaite faire tourner.

**`back-end:` et `front-end:`**
Noms logiques des services. Ils sont libres, mais doivent être uniques. Docker Compose les utilise ensuite comme **noms d'hôtes internes** : depuis le conteneur `front-end`, on peut joindre le back-end via `http://back-end:5252` (utile pour la Task 5 et au-delà).

**`build:`**
Indique à Compose comment **construire** l'image. C'est l'équivalent d'un `docker build`.

- `context: ./back-end` : dossier qui sert de contexte de build (ce sont les fichiers que le Dockerfile peut copier)
- `dockerfile: Dockerfile` : nom du fichier Dockerfile à utiliser dans ce contexte

**`image: softy-pinko-back-end:task4`**
Nom et tag à donner à l'image une fois construite. C'est l'équivalent du `-t` de `docker build`. Si l'image existe déjà localement avec ce nom, elle peut être réutilisée sans rebuild.

**`ports:`**
Liste des redirections de port, au format `"HÔTE:CONTENEUR"`.

- `"5252:5252"` : le port `5252` de la machine hôte est mappé sur le port `5252` du conteneur (Flask)
- `"9000:9000"` : pareil pour Nginx

> Attention au format : `"5252:5252"` est entre guillemets car YAML interprète sinon `5252:5252` comme une notation horaire en base 60. La forme `"hôte:conteneur"` est sans ambiguïté.

### À propos de `depends_on`

Bien que les mots-clés à connaître incluent `depends_on`, il n'est pas indispensable ici. `depends_on` sert à **ordonner** le démarrage : par exemple `front-end: depends_on: [back-end]` ferait démarrer le back-end **avant** le front-end. Dans notre cas, les deux services peuvent démarrer en parallèle sans problème, donc on omet cette directive.

## Construction des images

Depuis le dossier `task4/` :

```bash
docker-compose build
```

Compose lit `docker-compose.yml`, repère les deux services, et exécute en séquence l'équivalent de :

```bash
docker build -f ./back-end/Dockerfile -t softy-pinko-back-end:task4 ./back-end
docker build -f ./front-end/Dockerfile -t softy-pinko-front-end:task4 ./front-end
```

Les couches déjà mises en cache (depuis la Task 3) sont réutilisées : la construction est quasi instantanée.

## Lancement de l'application

```bash
docker-compose up
```

Cette commande :

1. Construit les images si elles n'existent pas (ou les utilise si déjà présentes)
2. Crée un **réseau Docker** dédié (visible dans les logs : `Network task4_default Created`)
3. Crée les conteneurs (`task4-back-end-1` et `task4-front-end-1`)
4. Les démarre **en parallèle**
5. **Attache** la sortie de tous les conteneurs au terminal courant

### Sortie attendue

```
[+] Running 3/3
 ⠿ Network task4_default        Created
 ⠿ Container task4-back-end-1   Created
 ⠿ Container task4-front-end-1  Created
Attaching to task4-back-end-1, task4-front-end-1
task4-back-end-1   |  * Serving Flask app 'api'
task4-back-end-1   |  * Running on http://172.18.0.2:5252
task4-front-end-1  | /docker-entrypoint.sh: Configuration complete; ready for start up
task4-front-end-1  | 2023/06/12 19:27:43 [notice] 1#1: nginx/1.25.0
task4-front-end-1  | 2023/06/12 19:27:43 [notice] 1#1: start worker processes
```

Les logs des deux conteneurs sont **préfixés** par leur nom (`task4-back-end-1` / `task4-front-end-1`), ce qui permet de les distinguer en un coup d'œil. Plus besoin de deux terminaux.

### Conventions de nommage

Compose nomme automatiquement les ressources en utilisant le nom du dossier comme préfixe :

- **Projet** : `task4` (nom du dossier)
- **Réseau** : `task4_default`
- **Conteneurs** : `task4-<service>-1` (le `-1` indique l'instance, utile en cas de scaling)

## Le réseau Docker créé par Compose

Compose crée automatiquement un **réseau bridge** privé pour l'application. À l'intérieur de ce réseau :

- Les conteneurs peuvent se joindre entre eux par leur **nom de service** (DNS interne)
- Le conteneur `front-end` peut résoudre `back-end` vers l'IP du conteneur back-end
- Aucune configuration manuelle n'est nécessaire

C'est un avantage majeur sur la Task 3 où il fallait s'en remettre au routage Docker par défaut. Cela ouvre la porte à utiliser des URLs internes comme `http://back-end:5252` au lieu de `http://localhost:5252`, ce qui sera exploité dans les tâches suivantes.

## Tester l'application

Ouvrir un navigateur sur [http://localhost:9000](http://localhost:9000) : le site doit s'afficher avec le titre dynamique `Hello, World!` injecté en haut, exactement comme dans la Task 3.

## Commandes utiles de Docker Compose

| Commande | Effet |
|----------|-------|
| `docker-compose build` | Construit ou reconstruit les images de tous les services |
| `docker-compose up` | Démarre tous les services au premier plan |
| `docker-compose up -d` | Démarre tous les services en arrière-plan (détaché) |
| `docker-compose down` | Arrête et supprime tous les conteneurs et le réseau |
| `docker-compose ps` | Liste les conteneurs gérés par Compose |
| `docker-compose logs <service>` | Affiche les logs d'un service spécifique |
| `docker-compose restart` | Redémarre tous les services |

Pour **arrêter proprement** l'application : `CTRL+C` dans le terminal où tourne `docker-compose up`, puis `docker-compose down` pour nettoyer.

## Concepts clés à retenir

- **Orchestration** : Docker Compose orchestre plusieurs conteneurs comme un tout cohérent. C'est le premier pas vers des outils plus puissants comme Kubernetes.
- **Configuration déclarative** : on **décrit l'état souhaité** (`docker-compose.yml`) au lieu d'enchaîner des commandes impératives. Le fichier est versionné, partageable, reproductible.
- **YAML** : format texte structuré par indentation. Très lisible mais sensible aux espaces (toujours utiliser des **espaces**, jamais des tabs).
- **Réseau implicite** : Compose crée un réseau privé où chaque service est joignable par son nom. C'est plus propre que de passer par `localhost` et les ports exposés.
- **Mapping de ports `"hôte:conteneur"`** : indispensable pour qu'on puisse accéder aux services depuis le navigateur. Si on n'expose qu'un port (sans mapping), il reste accessible aux autres conteneurs mais pas à l'hôte.
- **Une commande pour tout lancer** : `docker-compose up` remplace l'ensemble des `docker build` et `docker run` des tâches précédentes. C'est ce gain de simplicité qui justifie l'introduction de Compose.