---
title: "1.5 MultiStage Build"
weight: 5
sectionnumber: 1.5
---

Often you're going to use some kind of libraries, tools or dependencies during the build phase of your application that are not necessary during the runtime of the container.
To improve security and efficiency we only include whats absolutely necessary in the image. So we often remove these dependencies in the build phase after the application itself has been built.

In this lab you're going to learn how to use multistage builds and what they are good for.

## Purpose

If the application is not available as a prebuilt artifact, in many cases, the application itself gets built directly during the docker build process `docker build -t ...`

### Java Spring Boot Gradle build Example

The complete example can be found at <https://github.com/appuio/example-spring-boot-helloworld>.

```Dockerfile
FROM registry.access.redhat.com/ubi9/openjdk-17

LABEL org.opencontainers.image.authors="midcicd@puzzle.ch" \
      io.k8s.description="APPUiO Example Spring Boot App" \
      io.k8s.display-name="APPUiO Spring Boot App" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="springboot"

EXPOSE 8080 9000

RUN mkdir -p /tmp/src/
ADD . /tmp/src/

RUN cd /tmp/src && sh gradlew build --no-daemon

RUN ln -s /tmp/src/build/libs/springboots2idemo*.jar /deployments/springboots2idemo.jar

```

During the docker build the actual application source code is added to the context and built using the `gradlew build` command.
Gradle in this case is only used during the build phase, since it produces a jar that is then executed with `java -jar ...` at execution time.

Build phase dependencies:

* Java
* Gradle

Runtime phase dependencies:

* Java

## Multi-stage builds

With multistage builds you now have the possibility to actually split these two phases, so that you can pass the built artifact from phase one into the runtime phase, without the need of installing build time dependencies in the resulting docker image. Which means that the image will be smaller and consist of less unneeded dependencies.

Read more about Docker multi-stage builds at <https://docs.docker.com/develop/develop-images/multistage-build/>

## Create a multi-stage build

Turn the docker build from the first example (Java Spring boot <https://github.com/appuio/example-spring-boot-helloworld>) into a docker multistage build.
As a second image you can use `registry.access.redhat.com/ubi9/openjdk-17-runtime`. Try to find the solution before looking at it.

Please create two seperate images to see the actual size difference as well.

{{% details title="Show me the solution" %}}

We start by cloning the repository and building the orginal image:

```bash
cd /home/project
git clone https://github.com/appuio/example-spring-boot-helloworld.git
cd example-spring-boot-helloworld
docker build -t example-spring-boot-helloworld:v0.1 .
```

Now let us build the next version using an optimized Dockerfile, change the content of `Dockerfile` to the text below:

```Dockerfile
FROM registry.access.redhat.com/ubi9/openjdk-17 AS build

LABEL org.opencontainers.image.authors="noreply@acend.ch" \
      io.k8s.description="acend example spring boot app" \
      io.k8s.display-name="acend spring boot app" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="springboot"

RUN mkdir -p /tmp/src/
ADD . /tmp/src/

RUN cd /tmp/src && sh gradlew build --no-daemon

FROM registry.access.redhat.com/ubi9/openjdk-17-runtime

EXPOSE 8080 9000

COPY --from=build /tmp/src/build/libs/springboots2idemo*.jar /deployments/springboots2idemo.jar
```

Now build a new version of the image and compare the size:

```bash
docker build -t example-spring-boot-helloworld:v0.2 .
docker images
```

{{% /details %}}
