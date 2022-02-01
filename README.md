Ce readme est la synthèse globale du module, mais il y a des README pour les TPs dans les dossiers
# Docker
## Image
On utilise une image qui est le fichier de system et la configuration de l'applicationpour pour créer un container :
- Search for images in the store : ``docker search``
- Récupérer une image de Docker Store : ``docker pull image_name[:version]`` (un docker run fait un pull si pas en local)
- Voir les images en local : ``docker images``
- Know more about a specific image : ``docker inspect alpine``
### Dockerfile
- Base image : `FROM image_name[:version]`
- Run command (add a new layer of the image) : `RUN command`. Ex : 
    - Install modules : `RUN apk add --update module_name`
- Copy files in the image : ``COPY source dest``
- Specify a port : `EXPOSE port_number`
- Run command on startup of the container : `CMD [ "executable", args...]`
- Ajout de variable d'environement : `ENV NAMEVAR=... \ ...`
- Build : ``docker build [--no-cache] -t timlab74/app_name ./source``
## Run un container (instance d'une image)
- Run un container basé sur une image : ``docker run image_name command``
- Run an interacive termial with ``-it`` (useful for debug). Ex : ``docker run -it image_name command`` (quitter avec ``exit``)
- Voir les container en run : ``docker ps [-a]`` ou ``-a`` sers d'historique
- ``docker run --help``
- Run en detatched mode (détache le run du terminal): ``docker run -id image_name``
- Stop un run : ``docker rm -f container_name/id``. Combinaison de : 
    - ``docker stop container_id``
    - ``docker rm container_id``
- Assign a name : ``docker run --name container_name container_id``
- Pass env var : ``docker run -e VARNAME="..."``. Vars : 
    - AUTHOR
- Publier tout les port exposé du container a des ports random : ``-P``
- See running docker ports : ``docker port container_name``
- Set specific ports : ``-p container_port:host_port``
- Run a command in a running container : `docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`
- Copy files/floders between container and local filesystem : `docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH` ou `docker cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH|-`
## Network
- Créer un réseau : ``docker network create tp1-network``
- Lier as un container : Ajouter `--network=network_name` dans la command run
## Database
- Dans un container de base de données les scrpit sql mis dans le dossier `docker-entrypoint-initdb.d` seront exécutés dans l'ordre alphabétique
- Gérer la perte de données après la suppression d'un container via les volumes : ajout de ``-v /directory:/var/lib/postgresql/data``
# Git
## Inspection command
- Working directory changes details : ``git diff``
- Staging area infos : ``git status``
- Commit history :
    - Classic ``git log``
    - Prettier : ``git log --graph --decorate --pretty=oneline --abbrev-commit --all``
## First step
- Config : `git config --<where> user.<credentialType> <credential>`
    - Where :
        - `--local`
        - `--global`
    - Crendential Type : 
        - `user.name`
        - `user.email`
- Initialisation du repo :
    - `git init`
    - Ajout d'un fichier `README.md`
    - Ajout d'un fichier `.gitignore`
- Documentation : ``git <command> --help`` 
## Processus
- Ajout d'un fichier ou de tous dans la partie staged : ``git add <file/-A/.>``
- Commit des fichiers staged : ``git commit -m "Commentaire"
- Push sur origin (la branche remote) : `git push origin master`
## Basic commands
- Clone a project : `git clone <url_of_the_project>`
- Fetch distand modifs (sans les merge) : `git fetch -p`
- Fetch et les merge dans la branch : `git pull`
- Merge une branche dans la branche actuel : `git merge <branch_name>`
- Rebase la branche actuel sur une autre : `git rebase <branch_name>`
## Autre 
- Stash (mettre les modifs de coté pour faire un pull par exemple) : ``git stash``
- Rewrite history : `git rebase -i`

