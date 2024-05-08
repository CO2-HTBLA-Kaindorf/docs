# How to create a Quarkus Project 

# Alles lokal auf dem eigenen Laptop

## Create Quarkus Template Code

Hiermit wird ein einfaches `HelloWorld` Projekt für Quarkus erzeugt.

```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.8.2:create \
    -DprojectGroupId=at.schtl.co2mqtt \
    -DprojectArtifactId=co2mqtt-demo \
    -Dextensions='resteasy-reactive, smallrye-health'
```

Fehler können auftreten. Ich musste diese Zeile löschen.

* remove `<artifactId>quarkus-resteasy</artifactId>`

or use Quarkus (https://code.quarkus.io)

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```shell script
./mvnw compile quarkus:dev
```
or 

```shell script
./mvnw clean quarkus:dev
```

## Create uber-jar

What is a uber-jar. 

https://blog.payara.fish/what-is-a-java-uber-jar

How to create an uber-jar?

Precondition - add this entry to the `application.properties`
```bash
quarkus.package.type=uber-jar
```

and build the package with

```
./mvnw clean package
```
the result should this:

![image](https://github.com/CO2-HTBLA-Kaindorf/utils/assets/16894982/62bc2652-da70-4538-9781-b70f1f0828bd)

# Create a Docker Image

1. Wir brauchen ein Docker-File
Erzeugen wir ein Docker-File in:

![image](https://github.com/CO2-HTBLA-Kaindorf/utils/assets/16894982/e1974ac0-a08a-4add-ba3e-443fc899dd88)

mit folgenden Inhalt:
```bash
FROM eclipse-temurin:21-jre

RUN mkdir -p /opt/application
COPY *-runner.jar /opt/application/co2backend.jar
WORKDIR /opt/application
CMD [ "java", "-jar", "co2backend.jar" ]
```
und erzeugen/holen uns einen Docker-Container:

1. kopieren des Docker-File in den target Ordner des Projekts

```
cp src/main/docker/Dockerfile target/
```

2. Docker-Container mit unserer Application bauen lassen
 
```
docker build --tag ghcr.io/co2-htlbla-kaindorf/co2backend:latest ./target
```

`ghcr.io`: github docker repos; `co2-htlbla-kaindorf`: github-user oder github-organisation name

3. Checken, ob Docker-Image da ist:

```bash
docker image ls 
```
![image](https://github.com/CO2-HTBLA-Kaindorf/utils/assets/16894982/3c8ba5d7-3478-409e-a908-0f830ace5e00)

starten des Docker-Container mit
```
docker run <id>
```

Result

![image](https://github.com/CO2-HTBLA-Kaindorf/docs/assets/16894982/418db90d-3f57-4f99-a99a-7b03f195720f)

Nun haben wir unsere Quarkus Applikation im Docker-Container. Diesen Vorgang können wir mit jeder Application (auch Spring-Boot) erledigen.


# Add Database to Quarkus

Erweitern wir unser bestehendes Projekt um die Datenbank postgres. Hierzu ist es nötig das Projekt anzupassen:

```
./mvnw quarkus:add-extension -Dextensions='hibernate-orm-panache, jdbc-postgresql'
```

Folgende Ausgabe sollte erscheinen:

![image](https://github.com/CO2-HTBLA-Kaindorf/docs/assets/16894982/d141c4bb-3e07-4c12-9742-668ec5ef9958)

Es sollte die `pom.xmml` angepasst worden sein.

## Start postgres in Docker

Um die Datenbank zu starten benötigen wir einen Docker-Container. Dieser Container sollte aber nicht immer händisch gestartet werden müssen, sondern via einem Skript.
Wir erstellen im Quarkusprojekt das File `postgres-create.sh` mit folgendem Inhalt:

```
#!/bin/bash
set -e

DIR="./db-postgres"
if [ -d "$DIR" ]; then
  echo "${DIR} already exists, the installation has exited ..."
  exit 1
fi

echo "Installing postgres into ${DIR} ..."
mkdir ${DIR}
cd ${DIR}
cp ../src/main/docker/docker-compose-postgres.yml ./docker-compose-postgres.yml
```

und in 

![image](https://github.com/CO2-HTBLA-Kaindorf/docs/assets/16894982/9d8f867d-37a0-47ce-8f89-33e0b54bbd9f)

mit folgenden Inhalt

´´´
version: '3.1'

services:

  db:
    container_name: postgres
    image: postgres:15.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo
    ports:
      - 5432:5432
    volumes:
      - ./db-postgres/db:/var/lib/postgresql/data
      - ./db-postgres/import:/import
    networks:
      - postgres

#  adminer:
#    image: adminer
#    restart: always
#    ports:
#      - 8090:8080

# https://github.com/khezen/compose-postgres/blob/master/docker-compose.yml
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:6.20
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-pgadmin4@pgadmin.org}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-admin}
      PGADMIN_CONFIG_SERVER_MODE: 'False'
    volumes:
      - ./db-postgres/pgadmin:/root/.pgadmin
    ports:
      - 8090:80
    networks:
      - postgres
    restart: unless-stopped

networks:
  postgres:
    driver: bridge
´´´







