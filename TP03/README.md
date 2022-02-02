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
    
    ```
### Database
- Create role : `ansible-galaxy init ansible/roles/database`
- Delete the unused files
- Add the follwing lines in the tasks/main.yml : 
    ```
    
    ```
### Httpd
- Create role : `ansible-galaxy init ansible/roles/httpd`
- Delete the unused files
- Add the follwing lines in the tasks/main.yml : 
    ```
    
    ```
