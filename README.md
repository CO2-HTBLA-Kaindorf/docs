# utils

## Create Code
```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.8.2:create \
    -DprojectGroupId=at.schtl.co2mqtt \
    -DprojectArtifactId=co2mqtt-demo \
    -Dextensions='resteasy-reactive, smallrye-health'
```

Error can be:

* remove `<artifactId>quarkus-resteasy</artifactId>`

or use https://code.quarkus.io

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
2. Erzeugen wir ein Docker-File in:

![image](https://github.com/CO2-HTBLA-Kaindorf/utils/assets/16894982/e1974ac0-a08a-4add-ba3e-443fc899dd88)

mit folgenden Inhalt:
```bash
FROM eclipse-temurin:17-jre

RUN mkdir -p /opt/application
COPY *-runner.jar /opt/application/co2backend.jar
WORKDIR /opt/application
CMD [ "java", "-jar", "co2backend.jar" ]
```

