# FROM openjdk:11
# COPY Main.java .
# RUN javac Main.java
# CMD ["java", "Main"]

# Build
# Choisir l'image la version 3.6.3-jdk-11 de maven comme base image et la nommé myapp-build 
FROM maven:3.6.3-jdk-11 AS myapp-build
# Définir la variable d'environement MYAPP_HOME à /opt/myapp
ENV MYAPP_HOME /opt/myapp
# On ce place dans le répertoire /opt/myapp
WORKDIR $MYAPP_HOME
# On copy le pom.xml en local sur le container (à la racine de workdir du coup)
COPY simple-api/pom.xml .
# Idem pour le dossier src
COPY simple-api/src ./src
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