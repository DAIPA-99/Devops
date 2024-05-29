<img style="float: left; padding-right: 0px; width: 145px" src="https://upload.wikimedia.org/wikipedia/fr/thumb/e/e9/EPF_logo_2021.png/524px-EPF_logo_2021.png"> 
<br><br>


###  <div style="text-align: right">  Data Engineering - P2024 <br> <br>  <time datetime="2024-03-19">2024/05/29</time> <br> <br> <u>Name </u>: DAIPA Blandine </div>
<br> 



#   <center>  **DEVOPS** </center>


# TP 01 - Docker

## 1. Database

### Basics

To build the database image, use the following command:

```bash
docker build -t my-database .
```
And then I create my network using:
```bash
docker network create my-network
```
I run the admirer to o enable adminer/database communication
```bash
docker run --name adminer -p 8080:8080 --network my-network -d adminer
```
I run my database with the name #mypostgres with their informations like user, password, server and the port.
```bash
docker run --network my-network --name mypostgres -p 5432:5432 -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd my-database
```
#### Explanation:

* docker build: Builds the Docker image from the Dockerfile.
* docker network create: Creates a Docker network for communication between containers.
* docker run: Executes the Docker container for Adminer, a database administration tool, on port 8080.

#### Tips
To establish communication between your database and Adminer, you need to ensure that they are connected to the same Docker network. You can achieve this by executing the following command:
```bash
docker network connect app-network mypostgres
```
#### Dockerfile
We have created a Dockerfile with the following specifications:
```Dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/01-CreateScheme.sql
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/02-InsertData.sql

```
#### Explanation:

* We start with the base image postgres:14.1-alpine.
* We set environment variables for the PostgreSQL database: POSTGRES_DB, POSTGRES_USER, and POSTGRES_PASSWORD.
* We copy the SQL scripts 01-CreateScheme.sql and 02-InsertData.sql into the /docker-entrypoint-initdb.d/ directory within the container. These scripts will be executed automatically when the PostgreSQL container starts, initializing the database schema and inserting initial data.

## 2. Initializing the Database

### SQL Scripts
We have create .sql (01-CreateScheme.sql and 02-InsertData.sql)fichier in the same folder with Dockerfile
#01-CreateScheme.sql

```SQL
CREATE TABLE public.departments
(
 id      SERIAL      PRIMARY KEY,
 name    VARCHAR(20) NOT NULL
);

CREATE TABLE public.students
(
 id              SERIAL      PRIMARY KEY,
 department_id   INT         NOT NULL REFERENCES departments (id),
 first_name      VARCHAR(20) NOT NULL,
 last_name       VARCHAR(20) NOT NULL
);

```
#02-InsertData.sql

```SQL
INSERT INTO departments (name) VALUES ('IRC');
INSERT INTO departments (name) VALUES ('ETI');
INSERT INTO departments (name) VALUES ('CGP');

INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');

```
# Explanation :

* SQL scripts are executed during the creation of the Docker image to initialize the database with its structure and data.
* We need to rebuild the Docker image of our database to see our tables in Adminer.

## 3. Persist data

To be abble to do that we have run this command:

```bash
docker run -v /Users/daipablandine/Desktop/Devops/data:/var/lib/postgresql/data --name mypostgres -p 5432:5432 -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd my-database
```
# Explanation :

In this command:

* --name mypostgres gives a name to our container (here, "mypostgres").
* -v /Users/daipablandine/Desktop/Devops/data:/var/lib/postgresql/data specifies the volume. "/Users/daipablandine/Desktop/Devops/data" is the directory on our host machine where you want to store the database data, and "/var/lib/postgresql/data" is the directory inside the container where PostgreSQL stores its data.
* postgres:14.1-alpine is the Docker image we use for our PostgreSQL database, along with its specific version.

## 3. Backend API 

#Basics

To begin, we'll create a simple Java hello-world application and then proceed to build a Spring Boot application providing a simple API.

#Java Hello World:
In the file Main.java, we define a basic Java class to print "Hello World!" to the console:

```java
public class Main {

   public static void main(String[] args) {
       System.out.println("Hello World!");
   }
}
```
We compile this Java class using the following command:

```java
javac Main.java
```
#Dockerfile Configuration:

We configure the Dockerfile to handle both the build and run stages of our Spring Boot application.

```Dockerfile
# Build
FROM maven:3.9.6-amazoncorretto-21 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:21
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```
# Explanation :

* Build Stage (myapp-build):

** Uses a Maven-based image (maven:3.9.6-amazoncorretto-21) to compile the Spring Boot application.
** Copies the pom.xml file to resolve dependencies and downloads them offline.
** Copies the source code (src) and builds the application JAR without running tests.

* Run Stage:

** Switches to an Amazon Corretto JDK-based image (amazoncorretto:21).
** Copies the built JAR file from the build stage.
** Sets the command to run the Spring Boot application using java -jar myapp.jar.

To build my Java API, I use the following command:

```bash
docker build . -t java_api
```
This command builds a Docker image for my Java API using the current directory (.) as the build context and tags the image with the name java_api.

Then, to see "Hello World" printed in my terminal, I run the following command:

```bash
docker run java_api
```
This command runs a container from the java_api image, executing the Java application inside it. As a result, "Hello World" is displayed in the terminal output of the running container.

## Multistage build

In the directory /Users/daipablandine/Desktop/Devops/simpleapi/src/main/java/fr/takima/training/simpleapi/Controller, we create a file named GreetingController.java with the following specifications:

```bash
package fr.takima.training.simpleapi.Controller;

import org.springframework.web.bind.annotation.*;

import java.util.concurrent.atomic.AtomicLong;

@RestController
public class GreetingController {

   private static final String template = "Hello, %s!";
   private final AtomicLong counter = new AtomicLong();

   @GetMapping("/")
   public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
       return new Greeting(counter.incrementAndGet(), String.format(template, name));
   }

   record Greeting(long id, String content) {}

}
```
Next, we add a Dockerfile to the same directory to enable building and running the application.

Here's an example of how the Dockerfile might look:

```Dockerfile
# Build
FROM maven:3.9.6-amazoncorretto-21 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:21
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```
# Explanation :

This Dockerfile follows a multistage build approach:

* Stage 1 (build):

** Uses a Maven-based image to compile the Spring Boot application.
** Copies the project files and runs the Maven package command to build the application JAR.
** Stage 2 (runtime):

* Switches to a lightweight JDK image for runtime.

** Copies the built JAR file from the build stage.
** Exposes port 8080 for incoming connections.
** Specifies the command to run the application JAR file.
** With this Dockerfile in place, you can build and run the Spring Boot application using Docker.

After that, I build and run my simpleapi application using Docker with the following commands:

```bash
docker build -t simpleapi . 
docker run --name spring -d -p 8082:8080 simpleapi
```
These commands build the simpleapi Docker image and then run a container named spring from that image. The container is detached (-d), meaning it runs in the background, and port 8080 of the container is mapped to port 8082 on the host (-p 8082:8080).

When you open localhost, you should see the JSON response {"id":8,"content":"Hello, World!"}, indicating that the API is functioning correctly and responding with a greeting message.

# Answer of question:

A multistage build in Docker is used to optimize the Docker image size and improve build performance. It involves defining multiple build stages in a single Dockerfile, each serving a specific purpose.

## Backend API

TO do this we download the source code of simpleapi and then we configure the file application.yml like this:

```yml
spring:
  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        jdbc:
          lob:
            non_contextual_creation: true
    generate-ddl: false
    open-in-view: true
  datasource:
    url: jdbc:postgresql://mypostgres:5432/db
    username: usr
    password: pwd
    driver-class-name: org.postgresql.Driver
management:
  server:
    add-application-context-header: false
  endpoints:
    web:
      exposure:
        include: health,info,env,metrics,beans,configprops
```
Next, we proceed to build and run the Docker image:

```bash
docker build -t backendapi . 
docker run --name springapi -d -p 8082:8080 backendapi
```
With these endpoints  /departments/IRC/students on localhost, you can see this:

```JSON
[
  {
    "id": 1,
    "firstname": "Eli",
    "lastname": "Copter",
    "department": {
      "id": 1,
      "name": "IRC"
    }
  }
]
```
## 4. Http server

#Basics

Start by selecting a suitable base image for your HTTP server container. Create a basic landing page named index.html and place it inside your container. Additionally, create a Dockerfile with the following content:

```Dockerfile
# Utiliser l'image officielle httpd comme image de base
FROM httpd:2.4
# Copier index.html dans le répertoire de contenu web de Apache
COPY index.html /usr/local/apache2/htdocs/
COPY httpd.conf /usr/local/apache2/conf/httpd.conf
```
#Build and Run Docker

Build and run your image with the following commands:

```bash
docker build -t my-http-server .
docker run -d --network my-network --name my-http-container -p 8084:8080 my-http-server
```

#Customizing HTTPd Configuration

After configuring our httpd.conf, we add the following element to the end of the file located at /usr/local/apache2/conf/httpd.conf:

```Configuration
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://my-backend:8080/
ProxyPassReverse / http://my-backend:8080/
</VirtualHost>
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```
This configuration sets up the HTTP server as a reverse proxy to route requests to a backend API running at http://backendapi:8080/. Ensure that the backendapi hostname corresponds to the address of your backend API container within the Docker network.

# Answer of question:

A reverse proxy serves as an intermediary between clients and backend servers, efficiently distributing incoming web traffic to optimize performance and ensure high availability. It enhances security and privacy by shielding backend servers from direct exposure to external clients and offers SSL/TLS termination capabilities, simplifying certificate management and offloading decryption tasks. Additionally, reverse proxies improve performance by caching static content, reduce server load, and provide robust security features like content filtering and access control, fortifying defenses against malicious attacks and unauthorized access attempts.

## 5. Link application

To link our applications together using Docker Compose, we define a docker-compose.yml file with the following structure:

```YML
version: '3.7'

services:
    backend:
        container_name: my-backend
        build: 
            context: "/Users/daipablandine/Desktop/Devops/simple-api-student-main"
        networks:
            - my-network
        depends_on:
            - database

    database:
        build:
            context: "/Users/daipablandine/Desktop/Devops"
        container_name: my-database
        environment:
            POSTGRES_PASSWORD: pwd
        networks:
            - my-network
        volumes:
            - docker-volume:/var/lib/postgresql/data

    httpd:
        container_name: my-httpd
        build: 
            context: "/Users/daipablandine/Desktop/Devops/http"
        ports:
            - "80:80"
        networks:
            - my-network
        depends_on:
            - backend
            - database

networks:
    my-network:
volumes:
    docker-volume:
```
# Explanation:

In this configuration:

** We define three services: backend, database, and httpd.
** The backend service is built from a Dockerfile located in the specified context directory. It depends on the database service.
** The database service is built from a Dockerfile located in the specified context directory. It uses the latest PostgreSQL image and sets the POSTGRES_PASSWORD environment variable.
** The httpd service is built from a Dockerfile located in the specified context directory. It exposes port 80 and depends on both the backend and database services.
** All services are connected to the my-network netw

# Answer of question:

Docker Compose is essential because it simplifies the process of orchestrating multi-container Docker applications. It allows developers to define and manage all the services, networks, and volumes required for an application in a single YAML file, the docker-compose.yml. With Docker Compose, developers can easily spin up, scale, and manage complex applications with multiple interconnected containers using simple commands like docker-compose up, docker-compose down, and docker-compose build. This tool streamlines the development, testing, and deployment workflows, making it easier to collaborate on projects and ensuring consistency across different environments. Overall, Docker Compose significantly enhances productivity and efficiency in containerized application development and deployment.

## 6. Publish

With these command we will able to publish our images of our application : 

```bash
docker login
docker tag dockercompose-database dockercompose-database:1.0
docker push bdaipa/dockercompose-database:1.0

docker tag dockercompose-backend bdaipa/dockercompose-backend:1.0
docker push bdaipa/dockercompose-backend:1.0  

docker tag dockercompose-backend bdaipa/dockercompose-backend:1.0
docker push bdaipa/dockercompose-backend:1.0
```
# Link:
Link of my dockerhub:

```Link
https://hub.docker.com/repository/docker/bdaipa/my-database/general
```

# Answer of question:

We put our images into an online repository for several reasons:

__Accessibility__: Storing images in an online repository makes them easily accessible from anywhere with an internet connection, allowing for seamless sharing and distribution among team members or across different environments.

__Backup and Recovery__: Online repositories serve as a backup for Docker images, providing a secure and reliable storage solution. In case of system failures or data loss, images can be recovered from the repository, ensuring continuity in development and deployment workflows.

__Version Control__: Online repositories typically support versioning, enabling developers to track changes to images over time and roll back to previous versions if needed. This facilitates effective version control and collaboration in software development projects.

__Scalability__: Online repositories are designed to handle large volumes of image data and can scale to accommodate growing storage requirements. This scalability ensures that repositories can effectively manage an increasing number of images as projects evolve and expand.

__Integration with CI/CD Pipelines__: Many online repositories integrate seamlessly with continuous integration and continuous deployment (CI/CD) pipelines, allowing automated deployment of Docker images to various environments. This integration streamlines the development and deployment process, enabling faster release cycles and improved software delivery practices.

# TP 02 - Github Actions

## 1 Setup Github Actions
### 1.1 First steps into the CI World

For building and running our tests in our computer we have download maven in our computer computer using this link maven.apache.org and run these command to execute maven:

* First step we have dezip the file and put it into our Destop folder and then start to execute this command in our CLI

```bash
cd apache-maven
export PATH=/Users/daipablandine/Desktop/apache-maven/bin:$PATH
source ~/.bash_profile
mvn -version
```
And finally we have this result:




