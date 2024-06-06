Generate Quarkus project
```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.8.2:create
     -DprojectGroupId=at.htl.minikube
     -DprojectArtifactId=minikube
     -Dextensions='resteasy-reactive-jackson, smallrye-health'
```


# Build script for your project

```
#!/usr/bin/env bash

mvn -B package
cp src/main/docker/Dockerfile target/
docker login ghcr.io -u $GITHUB_ACTOR -p $GITHUB_TOKEN
docker build --tag ghcr.io/$GITHUB_REPOSITORY/backend:latest ./target
docker push ghcr.io/$GITHUB_REPOSITORY/backend:latest
```
