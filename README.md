# SPRING-BOOT-APPLICATION


## Technologies
``` 
Azure VM
Docker
GitHub
Jenkins
Kubernetes 
```


## Azure Vertual Machine Preparation

1- Create an azure virtual machine linux(ubuntu 20.04)

2- Connect to the azure virtual machine remotely using "MobaXterm"

3- Download spring boot application code then drag and drop it on MobaXterm

4- Install JDK.11 on azure vm

5- Install Docker on azure vm

6- Install Minikube on azure vm

7- Install Jenkins on azure vm



## Dockerize Spring Boot Project 


**create a Dockerfile (multi-stage build)**

The first stage have a gradle base image --- create a JAR file of our java project (gradle build)
Then the second stage have a java base image --- use this jar file to create our image (COPY --from=build ../*.jar ./spring-boot-application.jar)

```
FROM gradle:7.1.0-jdk11 AS build
RUN mkdir /home/gradle/src
COPY --chown=gradle:gradle . /home/gradle/src
WORKDIR /home/gradle/src
RUN gradle build --no-daemon

FROM openjdk:11-jre-slim

EXPOSE 8080

RUN mkdir /app

COPY --from=build /home/gradle/src/build/libs/demo-0.0.1-SNAPSHOT.jar /app/spring-boot-application.jar

ENTRYPOINT ["java", "-XX:+UnlockExperimentalVMOptions", "-Djava.security.egd=file:/dev/./urandom","-jar","/app/spring-boot-application.jar"]

```

**Create our spring boot app Image**

docker build --pull --rm -f "Dockerfile" -t demo:latest2 "."

![image](https://user-images.githubusercontent.com/61191521/203035996-076ab124-3ebd-4c1b-93c7-009ac2e1ec79.png)


**Add the image to docker registry**

 docker tag demo:latest2 ranahesham/springbootapp:v1.1
 
 docker image push ranahesham/springbootapp:v1.1
 
 ![image](https://user-images.githubusercontent.com/61191521/203037250-22a54755-ebe8-4d13-918d-26bac8331f9a.png)


**Containerize our spring boot app**

docker container run --name springbootapplication -d -p 80:80 ranahesham/springbootapp:v1.1

![image](https://user-images.githubusercontent.com/61191521/203036348-773aa244-7ebe-419f-8f66-7793a3f066ef.png)

![image](https://user-images.githubusercontent.com/61191521/203409635-fb2c6c2f-49ec-4c6b-a830-6785980ca0eb.png)


# Minikube

**Deploy Spring Boot App to local minikube**

![image](https://user-images.githubusercontent.com/61191521/203040513-21c80e6b-4694-400a-af15-3d57a04fc1a8.png)


**Dev deployment (Imperative configuration)**

kubectl create deployment --image=ranahesham/springbootapp:v1.1 dev --namespace=dev --replicas=3


**Prod deployment (Declarative configuration)**

kubectl create -f prod-deployments.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prod
  namespace: prod
spec:
  selector:
    matchLabels:
      app: prod-springboot
  replicas: 3
  template:
    metadata:
      labels:
        app: prod-springboot
    spec:
      containers:
        - name: demo-container
          image: ranahesham/springbootapp:v1.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
```


![image](https://user-images.githubusercontent.com/61191521/203043745-6d90d65b-7216-4490-998a-f6e8218b5e5f.png)
