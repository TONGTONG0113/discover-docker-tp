Réponses aux questions du TP

1-1. 
L’utilisation de l’option -e permet de séparer la configuration de l’image Docker. Les paramètres peuvent être modifiés au lancement du conteneur sans reconstruire l’image. Cela évite également d’inscrire directement les paramètres de configuration dans le Dockerfile. Pour les secrets sensibles, il est préférable d’utiliser un mécanisme dédié de gestion des secrets.

1-2. 
Un volume permet de conserver les données indépendamment du cycle de vie du conteneur. Si le conteneur est supprimé puis recréé, les données stockées dans le volume restent disponibles. Dans ce projet, postgres-data est monté dans /var/lib/postgresql/data.

1-3. 
Le Dockerfile utilise l’image postgres:17.2-alpine, configure le nom de la base, l’utilisateur et le mot de passe, puis copie les scripts SQL dans /docker-entrypoint-initdb.d/. Les commandes de construction, de lancement, de consultation des logs et d’accès à PostgreSQL sont présentées dans la section 3.

1-4. 
Le build multi-stage sépare la compilation de l’exécution. La première étape utilise Maven et le JDK 21 pour compiler le code et générer un fichier JAR. La seconde étape utilise une image JRE plus légère et copie uniquement le JAR. Les commandes WORKDIR, COPY, RUN, EXPOSE et CMD définissent respectivement le répertoire de travail, les fichiers à copier, la compilation, le port utilisé et la commande de démarrage.

1-5. 
Un reverse proxy reçoit les requêtes des clients et les transmet au backend. Il fournit un point d’entrée unique et permet de ne pas exposer directement le backend. Il peut également faciliter la configuration HTTPS, la répartition de charge et la gestion des requêtes.

1-6. 
Docker Compose permet de définir et d’orchestrer plusieurs conteneurs dans un fichier unique. Il centralise la configuration des services, des réseaux, des volumes, des dépendances et des ports. Il simplifie le démarrage, l’arrêt et la reconstruction de l’application.

1-7. 
Les commandes principales sont docker compose up -d pour démarrer les services, docker compose up --build -d pour reconstruire et démarrer, docker compose down pour arrêter et supprimer les conteneurs, docker compose ps pour vérifier leur état, docker compose logs pour consulter les logs et docker compose exec pour exécuter une commande dans un conteneur.

1-8. 
Le fichier docker-compose.yml définit les services database, backend et httpd. La base de données utilise un volume persistant et un contrôle de santé. Le backend attend que la base soit prête. Apache expose le port 8083 de la machine hôte et transmet les requêtes au backend. Les trois services communiquent via un réseau Docker. Les ports du backend et de PostgreSQL ne sont pas exposés à la machine hôte.

1-9. 
Les images sont taguées avec le nom du compte Docker Hub et une version, par exemple allgreenallright/docker-tp-database:1.0. La commande docker push publie chaque image sur Docker Hub. Les trois images prévues sont docker-tp-database:1.0, docker-tp-backend:1.0 et docker-tp-httpd:1.0.

1-10.
Un registre en ligne permet de partager les images avec les membres d’une équipe et de les utiliser sur d’autres machines sans reconstruire le projet à partir du code source. Il facilite la distribution des versions, le déploiement et l’intégration dans des pipelines CI/CD.

2-1.
Testcontainers est une bibliothèque Java qui permet de lancer des conteneurs Docker pendant les tests automatisés. Dans ce projet, un conteneur PostgreSQL est démarré afin de fournir une base de données pour tester les opérations de l’application. Cela permet d’exécuter des tests d’intégration dans un environnement proche de celui utilisé en production.

2-2
Les variables sécurisées, telles que les GitHub Actions Secrets, permettent de stocker des informations sensibles comme le nom d’utilisateur et le jeton d’accès Docker Hub. Elles permettent aux workflows d’utiliser ces informations sans les écrire directement dans le code source ni les exposer dans un dépôt public.

Dans ce projet, nous utilisons notamment les secrets suivants :
- DOCKERHUB_USERNAME
- DOCKERHUB_TOKEN
- SONAR_TOKEN

2-3
La configuration needs: test-backend garantit que le job de construction et de publication des images Docker ne démarre qu’après la réussite du job test-backend.

Sans cette dépendance, GitHub Actions peut exécuter les deux jobs en parallèle. Les images Docker pourraient alors être publiées même si les tests de l’application échouent.

2-4
La publication des images Docker sur Docker Hub permet de les stocker dans un registre distant. D’autres machines ou environnements peuvent ensuite télécharger et exécuter ces images sans devoir reconstruire l’application à partir du code source. Cela facilite la livraison et le déploiement de l’application.

Dans ce projet, les trois images publiées sont :
- allgreenallright/docker-tp-backend:latest
- allgreenallright/docker-tp-database:latest
- allgreenallright/docker-tp-httpd:latest

3-1

1. Configuration de l'inventaire

L'inventaire Ansible est défini dans le fichier `ansible/inventories/setup.yml`. Il indique les serveurs distants à administrer et les paramètres de connexion.

```yaml
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /path/to/private/key
    ansible_python_interpreter: /opt/docker_venv/bin/python
  children:
    prod:
      hosts:
        ruofan.zhao.takima.school:
```

- `all` regroupe tous les hôtes de l'inventaire.
- `vars` définit les variables communes aux hôtes.
- `ansible_user` indique l'utilisateur SSH utilisé pour la connexion.
- `ansible_ssh_private_key_file` indique le chemin de la clé privée SSH.
- `ansible_python_interpreter` définit l'interpréteur Python utilisé sur le serveur distant.
- `children` permet de regrouper les hôtes dans des groupes.
- `prod` représente le groupe des serveurs de production.
- `ruofan.zhao.takima.school` est le serveur distant administré.

La clé privée SSH doit rester confidentielle et ne doit pas être ajoutée au dépôt Git.

2. Vérification de la connexion

```bash
ansible all -i inventories/setup.yml -m ping
```

Cette commande utilise le module `ping` pour vérifier que le serveur est joignable et qu'Ansible peut exécuter des modules sur celui-ci. Le résultat `pong` confirme que la connexion fonctionne.

3. Collecte des informations système

```bash
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```

Le module `setup` collecte les informations du système distant, appelées *facts*. Elles permettent notamment d'adapter les tâches à la distribution Linux utilisée.

4. Gestion des paquets

```bash
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```

Cette commande utilise le module `apt` pour s'assurer que le paquet Apache2 est absent du serveur. L'option `--become` permet d'obtenir les privilèges nécessaires à la gestion des paquets système.

Ansible applique l'état demandé. Si Apache2 est déjà absent, une nouvelle exécution ne devrait pas modifier le système.


3-2

1. Structure des Playbooks

Un Playbook Ansible décrit les opérations à exécuter sur un ou plusieurs hôtes. Il est écrit au format YAML et contient notamment les hôtes cibles, les paramètres d'exécution et les rôles ou tâches à exécuter.

Le projet utilise plusieurs Playbooks indépendants :

- `docker-playbook.yml` : installation de Docker.
- `network-playbook.yml` : création du réseau Docker.
- `database-playbook.yml` : déploiement de PostgreSQL.
- `backend-playbook.yml` : déploiement de l'API Spring Boot.
- `httpd-playbook.yml` : déploiement du serveur HTTPD.
- `site.yml` : orchestration de l'ensemble des rôles.

2. Installation de Docker avec un Role

Les tâches d'installation de Docker sont regroupées dans `roles/docker/tasks/main.yml`. Elles comprennent :

1. L'installation des paquets nécessaires.
2. L'ajout de la clé GPG et du dépôt officiel Docker.
3. L'installation de Docker.
4. La création d'un environnement virtuel Python.
5. L'installation du SDK Docker pour Python.
6. Le démarrage du service Docker.

Le Playbook appelle ensuite le rôle :

```yaml
---
- hosts: all
  gather_facts: true
  become: true
  roles:
    - docker
```

Cette organisation sépare la logique d'installation du fichier principal et facilite la réutilisation des tâches.

3. Orchestration avec `site.yml`

Le fichier `site.yml` centralise le déploiement de l'application :

```yaml
---
- name: Deploy application infrastructure
  hosts: all
  become: true
  roles:
    - docker
    - network
    - database
    - backend
    - httpd
```

Les rôles sont exécutés dans l'ordre indiqué : installation de Docker, création du réseau, lancement de la base de données, déploiement du backend, puis lancement du proxy HTTPD.

4. Vérification des Playbooks

Avant l'exécution, la syntaxe peut être vérifiée avec :

```bash
ansible-playbook -i inventories/setup.yml site.yml --syntax-check
```

Le déploiement est lancé avec :

```bash
ansible-playbook -i inventories/setup.yml site.yml
```

Lors des vérifications du projet, l'exécution s'est terminée sans erreur. Une nouvelle exécution a renvoyé `changed=0` et `failed=0`, ce qui montre que les tâches exécutées n'avaient pas besoin de modifier l'état du serveur à ce moment-là.


3-3

1. Principe général

Le module `community.docker.docker_container` permet de gérer le cycle de vie des conteneurs Docker à partir d'Ansible. Il permet notamment de définir le nom du conteneur, l'image, l'état souhaité, la politique de redémarrage, les variables d'environnement, les volumes et les réseaux.

Dans ce projet, les conteneurs sont déployés avec des rôles distincts pour la base de données, l'API et le proxy HTTPD.

2. Déploiement de PostgreSQL

Le rôle `database` construit une image personnalisée nommée `student-database:latest`, puis démarre le conteneur PostgreSQL.

```yaml
- name: Create PostgreSQL database container
  community.docker.docker_container:
    name: database
    image: student-database:latest
    state: started
    restart_policy: unless-stopped
    env:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: "{{ postgres_password }}"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - name: app-network
```

- `name` définit le nom du conteneur.
- `image` indique l'image Docker utilisée.
- `state: started` demande que le conteneur soit démarré.
- `restart_policy: unless-stopped` permet son redémarrage automatique, sauf arrêt explicite.
- `env` définit le nom de la base, l'utilisateur et le mot de passe.
- `volumes` conserve les données PostgreSQL dans un volume Docker nommé `postgres-data`.
- `networks` connecte le conteneur au réseau `app-network`.

L'image personnalisée contient les scripts SQL d'initialisation. PostgreSQL ne les exécute automatiquement que lorsque le répertoire de données est initialisé pour la première fois.

3. Déploiement du backend

Le rôle `backend` construit l'image `simple-api-student:latest` à partir du Dockerfile du projet, puis démarre le conteneur :

```yaml
- name: Start backend container
  community.docker.docker_container:
    name: backend
    image: simple-api-student:latest
    state: started
    restart_policy: unless-stopped
    networks:
      - name: app-network
```

Le backend est connecté au même réseau que la base de données. Il utilise la configuration Spring Boot du projet pour se connecter à PostgreSQL via le nom de service `database`, sur le port `5432`.

Le Dockerfile du backend construit l'application avec Maven, puis exécute le fichier JAR dans une image Java d'exécution.

4. Déploiement du proxy HTTPD

Le rôle `httpd` construit l'image `student-httpd:latest` à partir du répertoire `httpd/`, puis démarre le conteneur :

```yaml
- name: Start HTTPD container
  community.docker.docker_container:
    name: httpd
    image: student-httpd:latest
    state: started
    restart_policy: unless-stopped
    ports:
      - "8083:80"
    networks:
      - name: app-network
```

- `ports: "8083:80"` publie le port 80 du conteneur sur le port 8083 du serveur.
- Le réseau `app-network` permet à HTTPD de communiquer avec le backend.
- La configuration Apache utilise `ProxyPass` et `ProxyPassReverse` pour transmettre les
