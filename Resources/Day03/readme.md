# Traditional Builds vs. Multi-Stage Builds in Docker

In a traditional Docker build, all steps are executed in a single sequence within one build container. This includes tasks like downloading dependencies, compiling code, and packaging the application. As a result, every layer created during this process ends up in the final image. While this method works, it often leads to large images that contain unnecessary components, increasing both the size and security risks of the image. This is where multi-stage builds become beneficial.

## What Are Multi-Stage Builds?

Multi-stage builds allow you to define multiple stages in your Dockerfile, each dedicated to a specific task. It's like running different parts of the build process in separate environments. By isolating the build environment from the final runtime environment, you can greatly reduce the size of the final image and minimize potential security vulnerabilities. This approach is particularly useful for applications with extensive build dependencies.

## Try It Out

1. **Download and Install Docker Desktop**:
   - Begin by downloading and installing Docker Desktop from the [official Docker website](https://www.docker.com/products/docker-desktop).

2. **Open a Pre-Initialized Project**:
   - Use [Spring Initializr](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.3.0-M3&packaging=jar&jvmVersion=21&groupId=com.example&artifactId=spring-boot-docker&name=spring-boot-docker&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.spring-boot-docker&dependencies=web), a quickstart generator for Spring projects, to create a project. It helps generate JVM-based projects for Java, Kotlin, and Groovy.

3. **Generate and Download the ZIP File**:
   - Select your project specifications (e.g., Maven build automation with Java, a Spring Web dependency, and Java 21), and click "Generate" to download the ZIP file.

4. **Unzip and Navigate the Project Directory**:
   - Unzip the downloaded file to see the following project structure:
     ```
     spring-boot-docker
     ├── Dockerfile
     ├── Dockerfile.multi
     ├── HELP.md
     ├── mvnw
     ├── mvnw.cmd
     ├── pom.xml
     └── src
         ├── main
         │   ├── java
         │   │   └── com
         │   │       └── example
         │   │           └── springbootdocker
         │   │               └── SpringBootDockerApplication.java
         │   └── resources
         │       ├── application.properties
         │       ├── static
         │       └── templates
         └── test
             └── java
                 └── com
                     └── example
                         └── springbootdocker
                             └── SpringBootDockerApplicationTests.java
     ```

### Creating a RESTful Web Service

Modify `SpringBootDockerApplication.java` under `src/main/java/com/example/springbootdocker/` with the provided Java code to create a simple Spring Boot web application that responds with "Hello World":

```java
package com.example.springbootdocker;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class SpringBootDockerApplication {

    @RequestMapping("/")
    public String home() {
        return "Hello World";
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDockerApplication.class, args);
    }
}

```

### Creating the Dockerfile

1. **Create a Dockerfile**:
   - Create a file named Dockerfile in the same folder that contains all the other folders and files (like src, pom.xml, etc.) and add the following content:
     ```dockerfile
     FROM eclipse-temurin:21.0.2_13-jdk-jammy
     WORKDIR /app
     COPY .mvn/ .mvn
     COPY mvnw pom.xml ./
     RUN ./mvnw dependency:go-offline
     COPY src ./src
     CMD ["./mvnw", "spring-boot:run"]
     ```

2. **Build the Docker Image**:
   - Execute the following command to build the Docker image:
     ```
     docker build -t spring-helloworld .
     ```

3. **Check the Image Size**:
   - Use the `docker images` command to check the image size:
     ```
     docker images
     ```
   - The output will show the image size, which might be large due to including the full JDK and Maven toolchain.

4. **Run the Application**:
   - Start a container using the built Docker image:
     ```
     docker run -d -p 8080:8080 spring-helloworld
     ```

## Using Multi-Stage Builds

### Create a Multi-Stage Dockerfile

1. **Create a Multi-Stage Dockerfile**:
   - Create a file named `Dockerfile.multi` in the project directory and add the following content:
     ```dockerfile
     FROM eclipse-temurin:21.0.2_13-jdk-jammy AS builder
     WORKDIR /opt/app
     COPY .mvn/ .mvn
     COPY mvnw pom.xml ./
     RUN ./mvnw dependency:go-offline
     COPY ./src ./src
     RUN ./mvnw clean install

     FROM eclipse-temurin:21.0.2_13-jre-jammy AS final
     WORKDIR /opt/app
     EXPOSE 8080
     COPY --from=builder /opt/app/target/*.jar /opt/app/*.jar
     ENTRYPOINT ["java", "-jar", "/opt/app/*.jar"]
     ```

2. **Build the Multi-Stage Image**:
   - Build the Docker image using the multi-stage Dockerfile:
     ```
     docker build -t spring-helloworld-builder -f Dockerfile.multi .
     ```

3. **Check the Optimized Image Size**:
   - Use the `docker images` command to compare the sizes:
     ```
     docker images
     ```
   - You'll get output similar to the following:
     ```
     REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
     spring-helloworld-builder  latest    c5c76cb815c0   24 minutes ago   428MB
     spring-helloworld          latest    ff708d5ee194   About an hour ago 880MB
     ```
     Your final image is just 428 MB, compared to the original build size of 880 MB. By optimizing each stage and only including what's necessary, multi-stage builds help you create smaller, more secure, and efficient Docker images. This approach not only improves performance but also makes your Docker images easier to manage and deploy.

## Docker Init Command

The `docker init` command is a useful utility that simplifies setting up a project to be built and run using Docker. When you execute `docker init` in your project directory, it guides you through creating essential files with sensible defaults tailored for your project:

- **.dockerignore**: Specifies files and directories to exclude from Docker builds.
- **Dockerfile**: Defines instructions for building a Docker image.
- **compose.yaml**: A Compose file for orchestrating multi-container applications.
- **README.Docker.md**: A README file specific to Docker.

Here’s how to use it:

1. Navigate to your project directory.
2. Run `docker init`.
3. Select the application platform your project uses (e.g., Go, Java, Node, Python, etc.).
4. Based on your selection, `docker init` generates the necessary files.
5. You can choose from various templates or start with a general-purpose setup.
6. After executing `docker init`, customize the created files according to your project’s specific requirements.

---

This comprehensive guide covers the basics of Dockerfile creation, multi-stage builds, and utilizing the `docker init` command to streamline Docker project setup. It provides a practical approach to building efficient Docker images and managing Docker projects effectively.
