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
