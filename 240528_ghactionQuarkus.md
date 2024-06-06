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

# Config github-actions - Pipeline

Das File muss in den Ordner:

![grafik](https://github.com/CO2-HTBLA-Kaindorf/docs/assets/16894982/a8c6ac73-b4e4-48d6-871b-062ef4d3ba5b)


```
name: Build and Deploy Dockerfiles
run-name: ${{ github.actor }} is building Docker images 🚀
on: [ push ]
jobs:
  build-images:
    permissions: write-all
    runs-on: ubuntu-22.04
    steps:
      - name: Check out repository code
        uses: actions/checkout@v4

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - run: |
          pwd
          ls -lah
        working-directory: ./k8s

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '21'
          cache: 'maven'

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build with Maven
        run: ./build.sh

```
