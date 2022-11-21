# SPRING-BOOT-APPLICATION


## Technologies
``` 
Azure Vertual Machine
Docker
GitHub
Jenkins
Kubernetes 
```


## Azure Vertual Machine Preparation

1- Create an azure virtual machine linux(ubuntu 20.04)

2- Connect to the azure virtual machine remotely using "MobaXterm"

3- Download spring boot application code then drag and drop it at MobaXterm

4- Install JDK.11 on azure vm

5- Install Docker on azure vm

6- Install Minikube on azure vm

7- Install Jenkins on azure vm



## Dokerize Spring Boot Project 


**create a Dockerfile (multi-stage build)**

The first stage have a gradle base image --- create a JAR file of our java project (gradle build)
Then the second stage have a java base image --- use this jar file to create our image (COPY --from=build ../*.jar ./spring-boot-application.jar)


**Create our spring boot app Image**

docker build --pull --rm -f "Dockerfile" -t demo:latest2 "."


**Containerize our spring boot app**

docker run --rm -d  -p 8080:8080/tcp demo:latest2



## Deploy Spring Boot App to local minikube


**Dev deployment (Imperative configuration)**

kubectl create deployments --image=demo:latest dev --namespace=dev --replicas=3 --dry-run -o yaml >dev_deployment


**Prod deployment (Declarative configuration)**

kubectl create -f prod_deployments.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prod-springboot
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
          image: demo:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
