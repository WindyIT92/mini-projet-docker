# **PayMyBuddy - Application de transactions financières**

### **Contexte**

*PayMyBuddy* est une application de gestion des transactions financières entre amis. Son infrastructure actuelle, fortement interconnectée et déployée manuellement, présente des inefficacités. Nous souhaitons améliorer son évolutivité et simplifier le processus de déploiement grâce à Docker et à l'orchestration de conteneurs.

### **Infrastructure**

L'infrastructure fonctionnera sur un serveur compatible Docker avec **Ubuntu 20.04** . Cette preuve de concept (POC) comprend la conteneurisation du backend Spring Boot et de la base de données MySQL, ainsi que l'automatisation du déploiement à l'aide de Docker Compose.

### **Composants :**

- **Côté serveur (Spring Boot) :** gère les données et les transactions des utilisateurs.
- **Base de données (MySQL) :** Stocke les utilisateurs, les transactions et les informations des comptes.
- **Orchestration :** Utilise Docker Compose pour gérer l'ensemble de la pile applicative.

### **Application**

*PayMyBuddy* se divise en deux services principaux :

1. **Service backend (Spring Boot) :**
    - Expose une API pour gérer les transactions et les interactions avec les utilisateurs
    - Se connecte à une base de données MySQL pour le stockage persistant.
2. **Service de base de données (MySQL) :**
    - Stocke les données utilisateur et transactionnelles
    - Le port 3306 est exposé pour permettre la connexion au serveur.

## **Application PayMyBuddy**

### **Aperçu**

L'application PayMyBuddy est une application Java qui utilise MySQL comme base de données. Ce projet utilise Docker et Docker Compose pour définir et orchestrer l'environnement.

### **Prérequis**

- Docker
- Docker Compose

### **Structure du dépôt**

- `Dockerfile`: Définit la configuration Docker pour la construction de l'image de l'application Java PayMyBuddy.
- `docker-compose.yml`: Orchestre la configuration de l'application PayMyBuddy et de sa base de données MySQL.

### **Configuration**

- L'application Java est configurée via des variables d'environnement pour se connecter à la base de données MySQL.
- Le service MySQL est préconfiguré avec un mot de passe root, une base de données et un mot de passe utilisateur. Le schéma et les données initiales sont chargés depuis le `./initdb` répertoire.

## **Conception et test de l’image**

### Création de l’image et paramétrage réseau

Dans un premier temps, vous allez créer l’application PayMyBuddy qui s’appuie sur Spring Boot & Maven :

1. Installez le JDk et maven sous la VM Ubuntu `sudo apt install openjdk-17-jre-headless maven`
2. Après s’être déplacer dans le dossier mini-projet-docker, on construit de l'application avec la commande  `mvn clean install`
3. En s’appuyant sur le fichier Dockerfile, créez l'image Docker pour PayMyBuddy avec **:**`docker build -t paymybuddy .`
4. Vérifiez que l’image est bien créer avec `docker images`
5. On crée par la suite un réseau pour nos futurs conteneurs avec `docker network create paymybuddynetwork`
6. Vérifiez que le réseau est bien créer avec `docker network ls`

### Création et initialisation de la base de données avec le conteneur MySql

L’application PayMyBuddy s’appuie sur une base de donnée MySql pour fonctionner. Et pour se faire, on va commencer par créer le 1er conteneur MySql avec cette commande : `docker run -d --name paymybuddydb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=db_paymybuddy --network paymybuddynetwork mysql:latest`

Une fois le conteneur MySql lancé, on voit procéder à son initialisation pour faire en sorte que la base de données de l’application soit prise en compte avec les étapes ci-dessous:

1. Se connecter au conteneur avec la commande `docker exec -it paymybuddydb /bin/bash`
2. Une fois dans le conteneur, se connecter à la base de données MySql avec `mysql -uroot -p` et mettre le mot de passe par la suite
3. Entrez `show databases;` pour être sur que la base de données de PayMyBuddy est pris en compte
4. Une fois qu’on a vu “**db_paymybuddy**” dans le tableau des bases de données détectées, on va donc le sélectionner avec la commande `USE db_paymybuddy`
5. Puis quitter MySql et le conteneur en tapant deux fois `exit`

### Premier test de l’application PayMyBuddy

Une fois que les étapes précédentes sont faites, il est enfin temps de lancer le conteneur de l’application PayMyBuddy avec cette commande : `docker run -d -p 8080:8080 --name paymybuddy --network paymybuddynetwork paymybuddy:v1 sleep infinity`

SI tout est bon, vous devriez pouvoir accéder à votre application depuis un navigateur via le port 8080 et obtenir ce résultat :

<img width="1494" height="824" alt="PayMybuddy proof 1" src="https://github.com/user-attachments/assets/001d7b1c-b4a5-4c3e-a82e-7fadd8736033" />


## **Orchestration avec Docker Compose**

Le fichier `docker-compose.yml`déploiera les deux services :

- **paymybuddy-backend :** Exécute l’application Spring Boot.
- **paymybuddy-db :** Base de données MySQL pour gérer les données et les transactions des utilisateurs.

On va dans un premier temps préparer l’environnement après que le premier test ait réussi, puis lancer le docker compose pour l’orchestration :

1. Pour se faire, on va dans un premier temps arrêter les conteneurs avec `docker stop`
2. Puis supprimer ces mêmes conteneurs avec `docker rm`
3. Etant donner que le fichier docker-compose.yml gère aussi la partie réseau, faut donc supprimer le réseau qu’on a crée précédemment avec la commande `docker network rm paymybuddynetwork`
4. Et on lance enfin la commande `docker compose up -d` pour démarrer tous les services en s’appuyant sur le fichier docker-compose.yml.

En attendant 10 secondes minimum, vous devrez de nouveau pouvoir accéder à l’application sur un navigateur via le port 8080 et obtenir ce résultat :

<img width="1077" height="683" alt="PayMybuddy proof 2" src="https://github.com/user-attachments/assets/348a0a05-b2e4-4887-be81-f346897e0c77" />


## Docker registry

Une fois que tout est bon, on va enregistré les images paymybuddy & MySQL dans le docker registry pour qu’ils soient réutilisable à tout moment et ensuite tester ces images dans le fichier docker-compose.yml pour s’assurer que tout a été pris compte.

Et pour se faire :

1. On lance d’abord cette commande pour lancer le conteneur qui s’occupe du registry : `docker run -d -p 5000:5000 --restart always --name registry --network mini-projet-docker_paymybuddynetwork registry:3`
2. On commence ensuite par taguer l’image PayMyBuddy avec cette commande : `docker tag mini-projet-docker-paymybuddy-backend:latest localhost:5000/paymybuddy`
3. On tague aussi l’image MySQL avec cette commande : `docker tag mysql:latest localhost:5000/mysql`
4. Vérifier que les images ont été crée avec `docker images`
5. Une fois ceci est fait, il reste plus qu’à pousser notre PayMyBuddy avec cette commande : `docker push localhost:5000/paymybuddy:latest`
6. Puis faire de même avec l’image MySQL avec cette commande : `docker push localhost:5000/mysql:latest`

Une fois que les images sont poussées, on utilisera dorénavant ces images dans le fichier docker-compose.yml en effectuant ces modifications ci-dessous:

1. Pour la partie PayMybuddy : 
`paymybuddy-backend:
    image: localhost:5000/paymybuddy:latest`
2. Pour la partie base de donnée :
`paymybuddydb:
    image: localhost:5000/mysql:latest`
3. Et relancer la commande `docker compose up -d`

Si les images du docker registry sont bel et bien prises en compte, vous devrez de nouveau pouvoir accéder à l’application sur un navigateur via le port 8080 et obtenir ce résultat :

<img width="1168" height="757" alt="PayMybuddy proof 3" src="https://github.com/user-attachments/assets/9c907de4-8096-4062-a81f-d4b019f2041c" />

