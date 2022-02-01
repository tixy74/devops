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
- Re build : ``docker build --no-cache -t timlab74/tp1/database ./database``
- Créer un network : ``docker network create tp1-network``
- Run database : ``docker run -d --network=tp1-network --name database -e POSTGRES_PASSWORD=admin timlab74/tp1/database``
- Run adminer : ``docker run -d --network=tp1-network --name database_view -p 8080:8080 adminer``
- Vérifié les deux container avec : ``docker ps`` qui donner : 
    ```
    CONTAINER ID   IMAGE                   COMMAND                  CREATED              STATUS              PORTS                    NAMES
    669ee43005b1   timlab74/tp1/database   "docker-entrypoint.s…"   About a minute ago   Up About a minute   5432/tcp                 database
    fb53f827212e   adminer                 "entrypoint.sh docke…"   5 minutes ago        Up 5 minutes        0.0.0.0:8080->8080/tcp   database_view
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
### Multistage build : Maven implementation
- Sur le site https://start.spring.io/, j'ai généré l'applicat