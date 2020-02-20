# WorkShop Rest API Python

WorkShop pour dévelloper sa première Rest API en python avec Docker.

## Partie 1 Installer Docker

__Note :__

Le tutoriel d'installation ne fonctionne que pour fedora 28 et plus.

D'abord mettez à jour vos packets avec :

`sudo dnf -y update`

Si des packets son mis à jour redémarrer votre pc.
Ensuite il faudra ajouter le repo docker à votre fedora :

`sudo dnf -y install dnf-plugins-core`

Puis set up le stable de Docker :

```
 sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
```

Maintenant que le repository est prêt, il faut installer le docker engine :

`sudo dnf install docker-ce docker-ce-cli containerd.io`

Puis le start :

`sudo systemctl enable --now docker`

Pour vérifier si docker est bien lancé :

`sudo systemctl status  docker`

Vous devriez voir apparaître :

```
Active: active (running)
```

Vous pouvez vérfier la version de votre Docker avec :

`sudo docker version`

Enfin pour vérifier si Docker fonctionne bien il vous suffit de pull une image avec :

`sudo docker pull alpine`


## Partie 2 Faire son premier Container

Nous allons voir comment créer un Docker Compose et un Dockerfile pour faire tourner nos containers.

Pour commencer nous allons créer un nouveau dossier. Dans celui-ci nous allons créer 4 fichiers, un app.py, un Dockerfile, un docker-compose.yml et enfin un requirements.txt.

Dans app.py il vous faudra copier coller ceci :

```
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
C'est ce qui nous permettra de faire tourner notre serveur.

Dans le requirements.txt il vous faudra juste ajouter :

```
flask
redis
```

Nous allons maintenant écrire notre Dockerfile avec:

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

Ce fichier dit à Docker de :

* Créer une image de Python 3.7
* Set le directory de travail à /code
* Set les variables d'environements de flask
* Installer les packets de gcc python
* Copier le requirements.txt et d'installer les dépendances de Python
* Copier le directory courant
* Set la commande par défaut pour le container.

Enfin nous allons voir comment définir notre service dans notre Compose.

Il vous suffit de copier dans votre `docker-compose.yml` :

```
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

Le compose va agir comme un "Makefile" et définir 2 services, Redis et Web.

Il ne vous reste plus qu'à démarrer votre service, pour cela rien de plus simple, il vous suffit de run :

`sudo docker-compose up`

Il vous suffit maintenant d'ouvrir votre navigateur et de coller :

`http://localhost:5000/`

Vous pouvez mainteant voir votre  Container/Compose tourner.