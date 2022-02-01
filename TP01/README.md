# TP 1
## Initialisation
Pull les trois images : 
```
docker pull httpd
docker pull postgres
docker pull openjdk:11
docker pull adminer
docker pull maven
```
## Database
Dans un dossier ``database`` : 
- Création d'un Dockerfile avec :
    ```
    FROM postgres:11.6-alpine
    ENV POSTGRES_DB=db \
    POSTGRES_USER=admin \
    ```
- Build : ``docker build -t timlab74/tp1/database ./database``
- Supprimer le container : `docker rm -f database`
- Re build : ``docker build --no-cache -t timlab74/tp1/database ./database``
- Créer un network : ``docker network create tp1-network``
- Run database : ``docker run -d --network=tp1-network --name database -e POSTGRES_PASSWORD=admin timlab74/tp1/database``
- Run adminer : ``docker run -d --network=tp1-network --name database_client -p 8080:8080 adminer``
- Vérifié les deux container avec : ``docker ps`` qui donner : 
    ```
    CONTAINER ID   IMAGE                   COMMAND                  CREATED              STATUS              PORTS                    NAMES
    669ee43005b1   timlab74/tp1/database   "docker-entrypoint.s…"   About a minute ago   Up About a minute   5432/tcp                 database
    fb53f827212e   adminer                 "entrypoint.sh docke…"   5 minutes ago        Up 5 minutes        0.0.0.0:8080->8080/tcp   database_client
    ```
- Scrpit d'initialisations :
    - Création d'un dossier scrpits dans lequel j'y ai mis :
        - ``01-CreateScheme.sql`` avec
            ````
            CREATE TABLE public.departments (
                id SERIAL PRIMARY KEY,
                name VARCHAR(20) NOT NULL
            );

            CREATE TABLE public.students (
                id SERIAL PRIMARY KEY,
                department_id INT NOT NULL REFERENCES departments (id),
                first_name VARCHAR(20) NOT NULL,
                last_name VARCHAR(20) NOT NULL
            );
            ```
        - ``02-InsertData.sql`` avec
            ```
            INSERT INTO departments (name) VALUES ('IRC');
            INSERT INTO departments (name) VALUES ('ETI');
            INSERT INTO departments (name) VALUES ('CGP');
            INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
            INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
            INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
            INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');
            ```
    - Ajout dans le Dockerfile de :
        ```
        COPY scrpits/01-CreateScheme.sql /docker-entrypoint-initdb.d
        COPY scrpits/02-InsertData.sql /docker-entrypoint-initdb.d
        ```
- Ce connecté au client sur http://localhost:8080
    - System : PostgresSQL
    - Server : database
    - Username : admin
    - Password : admin
    - Database : db
- Gérer le perte de données en cas de supression du container en ajoutant : ``-v /database:/var/lib/postgresql/data`` dans la command pour run
## Backend API
### Basics : Hello world with simple javac
Dans un dossier ``backend`` :
- Création d'un Main.java avec :
    ```
    public class Main {
        public static void main(String[] args) {
            System.out.println("Hello World!");
        }
    }
    ```
- Création d'un Dockerfile avec :
    ```
    FROM openjdk:11
    COPY Main.java .
    RUN javac Main.java
    CMD ["java", "Main"]
    ```
- Build : ``docker build -t timlab74/tp1/backend ./backend``
- Re build : ``docker build --no-cache -t timlab74/tp1/backend ./backend``
- Run backend : ``docker run --name backend timlab74/tp1/backend`` --> ``Hello world``
### Multistage build : Hello world avec maven
- Sur le site https://start.spring.io/, j'ai généré l'application avec les paramètre suivants :
    - Project: Maven
    - Language: Java 11
    - Spring Boot: 2.2.4
    - Packaging: Jar
    - Dependencies: Spring Web (for now)
- Dans le src/main/... j'ai créé un dossier controller dans lequel on crée un fichier ``GreetingController.java`` avec le code adéquoi
- Le dossier généré avec le controller ajouter a été mis dans le dossier backend
- Dans le dockerfile de backend, on change le code précédent par : 
    ```
    # Build
    # Choisir l'image la version 3.6.3-jdk-11 de maven comme base image et la nommé myapp-build 
    FROM maven:3.6.3-jdk-11 AS myapp-build
    # Définir la variable d'environement MYAPP_HOME à /opt/myapp
    ENV MYAPP_HOME /opt/myapp
    # On ce place dans le répertoire /opt/myapp
    WORKDIR $MYAPP_HOME
    # On copy le pom.xml en local sur le container (à la racine de workdir du coup)
    COPY simple-api-hello_world/pom.xml .
    # Idem pour le dossier src
    COPY simple-api-hello_world/src ./src
    # On exécute la commande suivante pour build le projet
    RUN mvn dependency:go-offline
    RUN mvn package -DskipTests

    # Run
    # Choisir la version 11-jre de openjdk pour base image
    FROM openjdk:11-jre
    # On redéfinit la variable et le workdir
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    # On copy le .jar dans target (dans le container) qu'on dans le répertoir nommé myapp.jar
    COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
    # On exécute la commande suivante au démarage du container pour run l'API
    ENTRYPOINT java -jar myapp.jar
    ```
- Re build : ``docker build --no-cache -t timlab74/tp1/backend ./backend``
- Run backend sur le port 8081 : ``docker run --name backend -p 8081:8080  timlab74/tp1/backend`` (vu que 8080 est déjà utilisé par la database_client). On as donc le résultat sur le lien http://localhost:8081 avec le rendu :
    ```
    {
    "id": 1,
    "content": "Hello, World!"
    }
    ``` 
    Où l'id s'incrémente à chaque refresh
### Backend API
- Dans le dossier backend, on met le dossier de simple-api récupérer sur git via la commande : `git clone https://github.com/Mathilde-lorrain/simple-api.git`
- On modifie le Dockerfile pour changer le path devant src et pom.xml (on enlève simple-api-hello_world)
- on modifie l'application.yml avec :
    ```
    url: jdbc:postgresql://database/db
    username: admin
    password: admin
    ```
- Re build : ``docker build --no-cache -t timlab74/tp1/backend ./backend``
- Run backend sur le port 8081 : ``docker run --name backend --network=tp1-network -p 8081:8080  timlab74/tp1/backend``
- On peut tester avec le lien http://localhost:8081/departments/IRC/students qui affiche : 
    ```
    [
        {
            "id": 1,
            "firstname": "Eli",
            "lastname": "Copter",
            "department": {
                "id": 1,
                "name": "IRC"
            }
        }
    ]
    ```
## Http server
### Hello world
Dans un dossier ``frontend``
- Création d'un index.html avec un h1 Hello world!
- Création d'un Dockerfile avec :
    ```
    FROM httpd
    COPY ./index.html /usr/local/apache2/htdocs/
    ```
- Build : ``docker build -t timlab74/tp1/frontend ./frontend``
- Re build : ``docker build --no-cache -t timlab74/tp1/frontend ./frontend``
- Run frontend : ``docker run --name frontend -p 80:80 timlab74/tp1/frontend``. On est sur le port 80 (port par défaut) alors si on va simplement sur le lien http://localhost, Hello world s'affiche.
### Configuration Proxy et reverse proxy
On met le reverse proxy pour bloqué des IP's en cas d'attaque
- Récupérer le httpd.conf avec : `docker cp frontend:/usr/local/apache2/conf/httpd.conf ./frontend`
- Y ajouter : 
    ```
    ServerName localhost
    <VirtualHost *:80>
        ProxyPreserveHost On
        ProxyPass / http://backend:8080/
        ProxyPassReverse / http://backend:8080/
    </VirtualHost>
    ```
    Et y décommenter les lignes :
    ```
    LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_http_module modules/mod_proxy_http.so
    ```
- On ajoute au Dockerfile la ligne : `COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf`, pour reprendre la fichier dans l'image
- Build avec la même command et run avec : `docker run --name frontend --network=tp1-network -p 80:80 -d timlab74/tp1/frontend`
## Docker compose
- Création du fichier ``docker-compose.yml`` à la racine du projet