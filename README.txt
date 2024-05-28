<img style="float: left; padding-right: 0px; width: 145px" src="https://upload.wikimedia.org/wikipedia/fr/thumb/e/e9/EPF_logo_2021.png/524px-EPF_logo_2021.png"> 
<br><br>


###  <div style="text-align: right">  Data Engineering - P2024 <br> <br>  <time datetime="2024-05-29">2024/03/19</time> <br> <br> <u>Name </u>: DAIPA Blandine </div>
<br> 



#   <center>  **DEVOPS** </center>


# Discover Docker

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