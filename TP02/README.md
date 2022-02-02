# Setup Github Actions
## CI
- Add a ``.github/workflows`` repository à la racine with a ``.main.yml`` inside. In this file we put :
    ```
    name: CI devops 2022 CPE
    on:
    #to begin you want to launch this job in main and develop
    push:
        branches: 
        - main
        - develop
    pull_request:

    jobs:
    test-backend:
        runs-on: ubuntu-18.04
        steps:
        #checkout your github code using actions/checkout@v2.3.3
        - uses: actions/checkout@v2.3.3
        
        #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
        - name: Set up JDK 11
            uses: actions/setup-java@v2
            with:
            distribution: 'zulu'
            java-version: '11'
        
        #finally build your app with the latest command
        - name: Build and test with Maven
            run: cd TP01/backend/simple-api && mvn clean verify
    ```
- Test it in the github repository in the menu action --> It's green ! :) 
## CD
- On https://github.com/tixy74/devops/settings/secrets/actions, add your credential as sectrets
- in the `.main.yml`, add in the jobs : 
    ```
    #define job to build and publish docker image
    build-and-push-docker-image:
        needs: test-backend
        # run only when code is compiling and tests are passing
        runs-on: ubuntu-latest
        # steps to perform in job
        steps:
        - name: Checkout code
            uses: actions/checkout@v2
        - name: Build image and push backend
            uses: docker/build-push-action@v2
            with:
            # relative path to the place where source code with Dockerfile is located
            context: ./TP01/backend
            # Note: tags has to be all lower-case
            tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-cpe-backend
        - name: Build image and push database
            uses: docker/build-push-action@v2
            with:
            # relative path to the place where source code with Dockerfile is located
            context: ./TP01/database
            # Note: tags has to be all lower-case
            tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-cpe-database
        - name: Build image and push httpd
            uses: docker/build-push-action@v2
            with:
            # relative path to the place where source code with Dockerfile is located
            context: ./TP01/httpd
            # Note: tags has to be all lower-case
            tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-cpe-httpd
    ```
- The `needs: test-backend` line is needed because we want git to build only if the test pass
- Test if in the github repository in the menu action --> Green again ! :D
## Publish
- Generate a token in https://hub.docker.com/settings/security
- L'ajouter dans les secrets sur Github
- Add before the Build image and pushs: 
    ```
    - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_TOKEN }}
    ```
- Dans les 3 Build image and pushs, ajouter respectivement 
    ```
     # build on feature branches, push only on main branch
    push: ${{ github.ref == 'refs/heads/main' }}
    ```
- On peux vérifié sur le lien https://hub.docker.com/repository/. Encore une fois, ça a fonctionné :DD
# Setup Quality Gate
- Create an organization in https://sonarcloud.io/create-organization
- Create a project
- Get both keys in https://sonarcloud.io/project/information?id=tixy74_devops
- Generate a token key for sonar here https://sonarcloud.io/account/security/
- Put it in Github secrets
- With those key, replace the mvn command with : `mvn -B verify sonar:sonar -Dsonar.projectKey=tixy74_devops -Dsonar.organization=tixy74 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./TP01/backend/simple-api/pom.xml`
- Generate github token with the link https://github.com/settings/tokens
- You don't need to put it in Github secrets, it is automatic
- Before the mvn command, add : 
    ```
    env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    ```
- In the website https://sonarcloud.io/project/new_code?id=tixy74_devops put `Previous version`
- Then when you commit and push, go check on https://sonarcloud.io/project/overview?id=tixy74_devops. Once again, this is green ! :DDD (smiley très content)
# Split pipeline
- Make a `.production.yml` in the workflows directory and put :
    ```
    name: CI devops 2022 CPE
    on:
      workflow_call:
          inputs:
            path:
              required: true
              type: string
            docker_name:
              required: true
              type: string
            
    jobs:
      build-and-push-docker-image:
        steps:
          - name: Build image and push backend
            uses: docker/build-push-action@v2
            with:
              # relative path to the place where source code with Dockerfile is located
              context: ${{inputs.path}}
              # Note: tags has to be all lower-case
              tags: ${{secrets.DOCKER_USERNAME}}/${{inputs.docker_name}}
              # build on feature branches, push only on main branch
              push: ${{ github.ref == 'refs/heads/main' }}
    ```
- Replace the 3 steps of the build and push docker images by 3 jobs using the .production.yml and add  a only: changes to do the job only if the concerned directory is changed : 
    ```
    #define job to build and publish docker image of backend
    build-and-push-backend:
      # run only when code is compiling and tests are passing
      needs: test-backend
      # run only when directory backend as any changes in it
      only:
        changes:
          - ./TP01/backend/**/*
      runs-on: ubuntu-latest
      # steps to perform in job
      steps:
        - uses: ./.github/workflows/.production.yml
          with:
            path: "./TP01/backend"
            docker_name: "tp-devops-cpe-backend"
      #define job to build and publish docker image of backend
    build-and-push-database:
      # run only when directory database as any changes in it
      only:
        changes:
          - ./TP01/database/**/*
      runs-on: ubuntu-latest
      # steps to perform in job
      steps:
        - uses: ./.github/workflows/.production.yml
          with:
            path: "./TP01/database"
            docker_name: "tp-devops-cpe-database"
      #define job to build and publish docker image of backend
    build-and-push-frontend:
      # run only when directory frontend as any changes in it
      only:
        changes:
          - ./TP01/frontend/**/*
      runs-on: ubuntu-latest
      # steps to perform in job
      steps:
        - uses: ./.github/workflows/.production.yml
          with:
            path: "./TP01/httpd"
            docker_name: "tp-devops-cpe-httpd"
    ```