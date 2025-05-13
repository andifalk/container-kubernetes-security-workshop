# 🧪 Ways to build Container Images

## 🎯 Objective

Learn different approaches on how to build Docker containers like Dockerfile, Google JIB and Buildpacks using a simple Java Spring Boot App.

---

## 🧰 Prerequisites

- Docker installed
- Java SDK (at least 17 LTS) installed
- Http Client installed ([Httpie](https://httpie.io/))

### Intallation

Install [Httpie](https://httpie.io/) as Http Client (more convenient as just curl):

```bash
sudo apt install httpie
```

---

## 🔹 Lab 1: Build an Run Spring Boot App

### Step 1: Checkout from GitHub

Clone the corresponding GitHub repository using:

```bash
git clone https://github.com/andifalk/kubernetes-sample-app.git
```

✅ **Expected:** You have a new folder called `kubernetes-sample-app`.

### Step 2: Build and Run the Spring Boot App

To build the Spring Boot App, change into the folder `kubernetes-sample-app` and then run:

```bash
./mvnw clean package
```

If the build finished successfully you can then run the application by:

```bash
./mvnw spring-boot:run
```

Open another terminal and try to access the provided APIs of this application:

```bash
http localhost:8080/api/hello
```

You can find all provided endpoints documented here: [API Endpoints](https://github.com/andifalk/kubernetes-sample-app?tab=readme-ov-file#api-endpoints)

### Step 3: Stop the Spring Boot App

To stop the Spring Boot App, in your first terminal just hit `Ctrl-C`.

---

## 🔹 Lab 2: Build Containers Image using Dockerfile

### Step 1: Inspect the provided Dockerfile

Have a look into the provided Dockerfile located in folder `docker/secure` using:

```bash
cat docker/secure/Dockerfile
````

The Dockerfile uses a base image with a Java 21 JRE and follows best security practice to run with a non-root user (UID 1002):

```Dockerfile
FROM bellsoft/liberica-openjre-alpine:21.0.6
ARG TARGETPLATFORM
RUN echo "I'm building for $TARGETPLATFORM"
COPY target/kubernetes-sample-app-1.0.0-SNAPSHOT.jar app.jar
EXPOSE 8080
RUN addgroup -S app -g 1002 && adduser -S appuser --u 1002 --g 1002
USER 1002
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### Step 2: Build Container Image using Dockerfile

```bash
docker build -f docker/secure/Dockerfile -t kubernetes-sample-app .
```

### Step 3: Run the Container Image (Build with Dockerfile)

```bash
docker run -p 8080:8080 kubernetes-sample-app
```

In the second terminal try again if the application is really running by calling:

```bash
http localhost:8080/api/hello
```

✅ **Expected:** You run the application as a container.

### Step 4: Stop the Container Image (Build with Dockerfile)

To stop the docker container from running just again type `Ctrl-C` in your first terminal.

To make sure no container is running you always can just perform

```bash
docker ps
```

---

## 🔹 Lab 3: Build Containers Image using Google JIB

### Step 1: Inspect the Maven pom.xml for JIB plugin

Have a look into the provided Maven build file located in folder `pom.xml` using:

```bash
cat pom.xml
````

The `pom.xml` file contains a plugin for the Google JIB plugin to build a container image based on [Google Distroless](https://github.com/GoogleContainerTools/distroless) base image:

```xml
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
    <version>3.4.5</version>
    <configuration>
        <to>
            <image>andifalk/kubernetes-sample-app:jib</image>
        </to>
        <from>
            <image>gcr.io/distroless/java17-debian12</image>
            <platforms>
                <platform>
                    <architecture>amd64</architecture>
                    <os>linux</os>
                </platform>
                <platform>
                    <architecture>arm64</architecture>
                    <os>linux</os>
                </platform>
            </platforms>
        </from>
        <container>
            <user>1002</user>
        </container>
    </configuration>
</plugin>
```

As you can see it uses `gcr.io/distroless/java17-debian12` as base image and also runs with a non-root user (UID 1002).

### Step 2: Build Container Image using JIB

```bash
./mvnw jib:dockerBuild -Dimage=kubernetes-sample-app
```

After build has finished you may have a look inside the image using

```bash
docker inspect kubernetes-sample-app
```

E.g. look for jib as a proof that this is really the image build by JIB.

### Step 3: Run the Container Image (Build with JIB)

As before just run this container again using:

```bash
docker run -p 8080:8080 kubernetes-sample-app
```

In the second terminal try again if the application is really running by calling:

```bash
http localhost:8080/api/hello
```

✅ **Expected:** You run the application as a container.

### Step 4: Stop the Container Image (Build with JIB)

To stop the docker container from running just again type `Ctrl-C` in your first terminal.

To make sure no container is running you always can just perform

```bash
docker ps
```

---

## 🔹 Lab 4: Build Containers Image using Buildpack (Paketo)

### Step 1: Build Container Image using Buildpack

[Cloud Native Buildpacks (The standardization)](https://buildpacks.io) and [Paketo (one implementation)](https://paketo.io/) are the default mechanism in Spring Boot to build container images using the Spring Boot Maven plugin.

So to build the container image just perform:

```bash
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=kubernetes-sample-app
```

After build has finished you may have a look inside the image using

```bash
docker inspect kubernetes-sample-app
```

E.g. look for buildpack as a proof that this is really the image build by Spring Boot and Paketo Buildpacks.

### Step 3: Run the Container Image (Build with Buildpack)

As before just run this container again using:

```bash
docker run -p 8080:8080 kubernetes-sample-app
```

In the second terminal try again if the application is really running by calling:

```bash
http localhost:8080/api/hello
```

✅ **Expected:** You run the application as a container.

### Step 4: Stop the Container Image (Build with Buildpack)

To stop the docker container from running just again type `Ctrl-C` in your first terminal.

To make sure no container is running you always can just perform

```bash
docker ps
```

---

## ✅ Wrap-Up

- ✅ Different approaches to build container images
- ✅ Run as non-root
- ✅ Using Dockerfile
- ✅ Using Google JIB and Distroless
- ✅ Using Spring Boot and Paketo Buildpack
- ✅ Run the Container and test the API provided by the application

---
