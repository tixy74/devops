# Inventories
- Create a file and folders : `ansible/inventories/setup.yml` and put in : 
    ```
    all:
      vars:
        ansible_user: centos
        ansible_ssh_private_key_file: ./id_rsa
      children:
        prod:
          hosts: timothee.labrosse.takima.cloud
    ```
- Then you can ping with : `ansible all -i ansible/inventories/setup.yml -m ping`
- Get OS distribution : `ansible all -i ansible/inventories/setup.yml -m setup -a "filter=ansible_distribution*"`
- Remove apache : `ansible all -i ansible/inventories/setup.yml -m yum -a "name=httpd state=absent" --become`
# Playbooks
- Create a `playbook.yml` file in the ansible directory and put :
    ```
    - hosts: all
      gather_facts: false
      become: yes
      tasks:
        - name: Test connection
          ping:
    ```
- Execute it with : `ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml`
## Roles
- Create a docker role with : `ansible-galaxy init ansible/roles/docker`
- Delete all the directory except tasks and handlers
- In the `-host` of the play book, to call the roles, add :
    ```
    roles:
      - docker
    ```
- Cut / paste the task from the play book into the main.yml of the dockers role tasks (without the "tasks:" title).
## Deploy the app
### Backend
- Create role : `ansible-galaxy init ansible/roles/backend`
- Delete the unused files
- Add the follwing lines in the tasks/main.yml : 
    ```
    - name: Run Backend
      docker_container:
        name: backend
        image: timlab74/tp-devops-cpe-backend
      networks:
        - name : network_one
    ```
- Add : `- backend` to the roles in the playbook.yml
### Database
- Create role : `ansible-galaxy init ansible/roles/database`
- Delete the unused files
- Add the follwing lines in the tasks/main.yml : 
    ```
    - name: Run Database
      docker_container:
        name: database
        image: timlab74/tp-devops-cpe-database
      networks:
        - name : network_one
      volumes:
        - /database:/var/lib/postgresql/data
    ```
- Add : `- database` to the roles in the playbook.yml
### Httpd
- Create role : `ansible-galaxy init ansible/roles/httpd`
- Delete the unused files
- Add the follwing lines in the tasks/main.yml : 
    ```
    - name: Run HTTPD
      docker_container:
        name: httpd
        image: timlab74/tp-devops-cpe-httpd
      networks:
        - name : network_one
      ports:
        - 80:80
    ```
- Add : `- httpd` to the roles in the playbook.yml
### Network
- Create role : `ansible-galaxy init ansible/roles/network`
- Delete the unused files
- Add the follwing lines in the tasks/main.yml : 
    ```
    - name: Create a network
      docker_network:
        name: network_one
    ```
- Add : `- network` to the roles in the playbook.yml
### Verif
- You can now check via the links
    - http://timothee.labrosse.takima.cloud/departments/IRC/students
    - http://timothee.labrosse.takima.cloud/
# Front
- Clone the git `https://github.com/Mathilde-lorrain/devops-front` and put it in the TP03 folder
- To the main workflow add for git action to push de new docker image frontend
  ```
  - name: Build image and push frontend
    uses: docker/build-push-action@v2
    with:
      # relative path to the place where source code with Dockerfile is located
      context: ./TP03/devops-front
      # Note: tags has to be all lower-case
      tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-cpe-frontend
      # build on feature branches, push only on main branch
      push: ${{ github.ref == 'refs/heads/main' }}
  ```
- Create role : `ansible-galaxy init ansible/roles/frontend`
- Delete the unused files
- Add the follwing lines in the tasks/main.yml : 
    ```
    - name: Run Frontend
      docker_container:
        name: frontend
        image: timlab74/tp-devops-cpe-frontend
      networks:
        - name : network_one
      ports:
        - 80:80
    ```
- Add : `- frontend` to the roles in the playbook.yml
- Change httpd.conf for that : 
  ```
  ServerName localhost
  <VirtualHost *:80>
      ProxyPreserveHost On
      ProxyPass /api http://backend:8080/
      ProxyPassReverse /api http://backend:8080/
      ProxyPass / http://frontend:80/
      ProxyPassReverse / http://frontend:80/
  </VirtualHost>
  ```
  So localhost/api redirects to the backend, and localhost to the front end
- Change the `.env.production` with that : `VUE_APP_API_URL=localhost/api`
- Publish on the domain with : `ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml`
- Change the frontend .env.production for : `VUE_APP_API_URL=timothee.labrosse.takima.cloud/api`
- Check if it works on http://timothee.labrosse.takima.cloud and http://timothee.labrosse.takima.cloud/api. It works ! 
# Continuous deployment