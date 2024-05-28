Report TP1

bash

docker volume create postgresdata

Run a PostgreSQL container with the created volume:

bash

docker run -d --name mypostgresqlguigui --network app-network -v postgresdata:/var/lib/postgresql/data my_postgres:v1

Build the custom PostgreSQL Docker image:

bash

docker build -t my_postgres:v1 .

Run Adminer for database management:

bash

    docker run -p 8090:8080 --network app-network --name=adminer -d adminer

Dockerfile for PostgreSQL

Dockerfile

FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY CreateScheme.sql /docker-entrypoint-initdb.d/
COPY InsertData.sql /docker-entrypoint-initdb.d/

Explanation

A multistage build in Docker is useful for optimizing the final size of the Docker image by eliminating unnecessary build dependencies and retaining only the artifacts required for running the application.
Goals and Good Practices

    Document each step of the process.
    Create an appropriate file structure with one folder per image.
    Develop a 3-tier web API application.

3-Tier Application Structure

    HTTP server
    Backend API
    Database

Database Setup

    Base image: postgres:14.1-alpine
    Initialize the database with SQL scripts.

SQL Scripts

01-CreateScheme.sql

sql

CREATE TABLE public.departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(20) NOT NULL
);

CREATE TABLE public.students (
    id SERIAL PRIMARY KEY,
    department_id INT NOT NULL REFERENCES departments(id),
    first_name VARCHAR(20) NOT NULL,
    last_name VARCHAR(20) NOT NULL
);

02-InsertData.sql

sql

INSERT INTO departments (name) VALUES ('IRC'), ('ETI'), ('CGP');

INSERT INTO students (department_id, first_name, last_name) VALUES
(1, 'Eli', 'Copter'),
(2, 'Emma', 'Carena'),
(2, 'Jack', 'Uzzi'),
(3, 'Aude', 'Javel');

Network Setup

Create a Docker network for container communication:

bash

docker network create app-network

Backend API
Basic Java Setup

    Compile the Java class:

    bash

javac Main.java

Dockerfile for running Java application:

Dockerfile

    FROM eclipse-temurin:17-jre-alpine
    COPY Main.class .
    CMD ["java", "Main"]

    Run the container and verify the output.

Multistage Build for Java

Dockerfile

Dockerfile

FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /usr/src/app
COPY . .
RUN javac Main.java

FROM eclipse-temurin:17-jre-alpine
WORKDIR /usr/src/app
COPY --from=builder /usr/src/app/Main.class .
CMD ["java", "Main"]

Spring Boot Application

    Generate a Spring Boot project on Spring Initializer.

    Create a simple GreetingController:

    java

package fr.takima.training.simpleapi.controller;

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

Dockerfile for the Spring Boot application:

Dockerfile

    FROM maven:3.8.6-amazoncorretto-17 AS build
    WORKDIR /app
    COPY pom.xml .
    COPY src ./src
    RUN mvn package -DskipTests

    FROM amazoncorretto:17
    WORKDIR /app
    COPY --from=build /app/target/*.jar app.jar
    ENTRYPOINT ["java", "-jar", "app.jar"]

Explanation of Multistage Build

A multistage build allows separating the build environment from the runtime environment, resulting in smaller and more secure Docker images.
HTTP Server

    Create a simple landing page index.html and a Dockerfile:

    Dockerfile

FROM httpd:2.4
COPY ./index.html /usr/local/apache2/htdocs/

Configure Apache as a reverse proxy:

apache

    ServerName localhost

    <VirtualHost *:80>
        ProxyPreserveHost On
        ProxyPass / http://YOUR_BACKEND_LINK:8080/
        ProxyPassReverse / http://YOUR_BACKEND_LINK:8080/
    </VirtualHost>
    LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_http_module modules/mod_proxy_http.so

Docker Compose

Create a docker-compose.yml to orchestrate the containers:

yaml

version: '3.7'

services:
  backend:
    build: ./backend
    networks:
      - my-network
    depends_on:
      - database

  database:
    build: ./database
    networks:
      - my-network

  httpd:
    build: ./httpd
    ports:
      - "80:80"
    networks:
      - my-network
    depends_on:
      - backend

networks:
  my-network:

Important Docker Compose Commands

    docker-compose up: Start all services.
    docker-compose down: Stop and remove all services.
    docker-compose build: Build or rebuild services.

Publishing Docker Images

    Login to Docker Hub:

    bash

docker login

Tag your images:

bash

docker tag my-database USERNAME/my-database:1.0

Push your images to Docker Hub:

bash

    docker push USERNAME/my-database:1.0

Tips and Considerations

    Environment Variables: It's better to set sensitive information like passwords using environment variables at runtime with the -e flag.
    Persistence: Use Docker volumes to persist data and avoid data loss when containers are removed.
    Reverse Proxy: Use a reverse proxy to handle requests and forward them to appropriate backend services.
    Docker Compose: Simplifies management of multi-container Docker applications.

This README provides a comprehensive guide for setting up and running a 3-tier application using Docker, covering database initialization, backend API creation, HTTP server configuration, and container orchestration with Docker Compose.