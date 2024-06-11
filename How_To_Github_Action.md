# How to create Github Action for Quarkus with Maven

### Step 1: Add uber-jar setup to application.properties

__src > resources > application.properties__:

```quarkus.package.jar.type=uber-jar```

Now you can try to build the package with

```mvn clean package```.

P.S. If mvn cmdlet not found exception appears, have fun downloading it :*)

### Step 2: Create Docker setup for you project

__src > main > docker__:

Create the file `Dockerfile` with the following content:

```docker
FROM eclipse-temurin:21-jre

RUN mkdir -p /opt/application
COPY *-runner.jar /opt/application/<name of your project>.jar
WORKDIR /opt/application
CMD [ "java", "-jar", "<name of your project>.jar" ]
```

### Step 3 (optional): Add Database to Quarkus

mvn quarkus:add-extension -Dextensions='hibernate-orm-panache, jdbc-postgresql'

This command will adjust the `pom.xml` acordingly.

#### Step 3.1: Start postgres via Docker

To be able to start your postgres database automaticly you need to create a script:

__src > main > docker__:

Create the file `postgres-create.sh` with the following content:

```yaml
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

Now also create the file `docker-compose-postgres.yaml` with the following content:

```yaml
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
```

### Step 4: Configure the Github Action

__Github > your Repository > Actions > new Workflow__:

Here you enter the following:

```yaml
name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        distribution: 'temurin' 
        java-version: '17'

    - name: Cache Maven repository
      uses: actions/cache@v2
      with:
        path: ~/.m2
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: |
          ${{ runner.os }}-maven-

    - name: Install dependencies
      run: mvn dependency:resolve

    - name: Build with Maven
      run: mvn clean package

    - name: Copy DockerFile to ./target
      run: cp src/main/docker/Dockerfile target/

    - name: List files in target directory
      run: ls -la target

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Log in to GitHub Container Registry
      run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

    - name: Build the Docker image
      run: docker build --tag ghcr.io/<username or organization>/<package_name>:latest ./target

    - name: Push the Docker image
      run: docker push ghcr.io/<username or organization>/<package_name>:latest

```
**Important**
  
* Set the correct Java version. In this case Temurin 17 is in use
* Fill in the correct, lowercase username / organization for the placeholders
* Set package_name (e.g. the Repository name)


### Now you should be all set :)
