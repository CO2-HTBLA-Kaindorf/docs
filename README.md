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
