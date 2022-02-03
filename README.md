Ce readme est la synthèse globale du module (principalement les TD et le récap de toute les commandes), mais il y a des README pour les TPs dans les dossiers TPXX
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
## Docker compose
- Dans un fichier docker-compose.yml on y met : 
    ```
    version: '3.7'
    services:
        backend:
            build:
                <./DockerFile_rep>
            networks:
                - <network_name>
            depends_on:
                - <container_name>
            volumes:
                - /<DockerFile_rep>:/<Save_rep>
             ports:
                - <container_port>:<host_port>
        networks:
            <network_name>:
    ```
- On execute avec : `docker-compose up`
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
- Verionnée : `git tag XXX` et `git push tag`
# Ansible
## SSH connection :
- Changer les droits pour la clé : `chmod 400 <path_to_your_key>`
- Connecter au SSH server : `ssh -i <path_to_your_key> centos@<your_server_domain_name>`
- Fermé le terminal ssh : `exit`
## Ansible connection :
- Modifié le fichier ``/etc/ansible/hosts`` et y ajouter : `centos@timothee.labrosse.takima.cloud` (`<distribution>@<server_domain_name>`)
- Ping the server : 
    - Avec la clé : `ansible all -m ping --private-key=<path_to_key> -u centos`
    - Avec l'inventory : `ansible all -i ansible/inventories/setup.yml -m ping`
- Get OS distribution : `ansible all -i ansible/inventories/setup.yml -m setup -a "filter=ansible_distribution*"`
## Setup Apache Server
- Install apache : ``ansible all -m yum -a "name=httpd state=present" --private-key=./id_rsa -u centos --become`` where `./id_rsa` is the path to your key
- Create an HTML page : `ansible all -m shell -a 'echo "<html><h1>Hello CPE</h1></html>" >> /var/www/html/index.html' --private-key=./id_rsa -u centos --become`
- Start Apache service : `ansible all -m service -a "name=httpd state=started" --private-key=./id_rsa -u centos --become`
- Remove it : `ansible all -i ansible/inventories/setup.yml -m yum -a "name=httpd state=absent" --become`
## Inventory
- Dans un fichier `ansible/inventories/setup.yml` mettre : 
    ```
    all:
      vars:
        ansible_user: centos
        ansible_ssh_private_key_file: <path_to_key>
      children:
        prod:
          hosts: domain_name or ip
    ```
## Playbook
- Dans un fichier `playbook.yml` dans le dossier ansible mettre :
    ```
    - hosts: all
      gather_facts: false
      become: yes
      tasks:
        - name: <taks_name>
          <task>:
    ```
    Task exemples : 
    - `ping`
    - 
    ```
        command: 
            cmd: ...
    ```
    - `dnf`
- Execute with `ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml`
### Role
- Create a docker role with : `ansible-galaxy init ansible/roles/docker`