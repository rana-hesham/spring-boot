# SPRING BOOT APPLICATION


## Technologies
``` 
Azure VM - Docker - GitHub - SonarQube - Jenkins - Kubernetes 
```


## Azure Virtual Machine Preparation

1- Create an azure virtual machine linux(ubuntu 20.04)

2- Connect to the vm remotely using "MobaXterm"

3- Download the spring boot application code then drag and drop it on MobaXterm

4- Install JDK.11 on azure vm

5- Install Docker on azure vm

6- Install Minikube on azure vm

7- Install Jenkins on azure vm



## Dockerize Spring Boot App 


**create a Dockerfile (multi-stage build)**

The first stage have a gradle base image ---> create a JAR file of our java project (gradle build)

Then the second stage have a java base image ---> Copy the output JAR file from the first stage (COPY --from=build ../*.jar ./spring-boot-application.jar)

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



## Jenkins Multibranch Pipeline

**Steps**

**1. Create a Jenkinsfile**

In Dev Branch with a 5 stages :

Lint Stage - Unit Test Stage - SonarQube Stage - Build Stage - Dev Depolyment Stage

```
pipeline {
    agent any
    stages {
        stage(lint) {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew lint'
            }
        }
        stage(unit_test) {
            steps {
                sh './gradlew test'
            }
        }
        stage(sonar_qube) {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "./gradlew sonarqube"
                }
            }
        }
        stage(build) {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'username', passwordVariable: 'pass')]) {
                sh 'docker login -u ${username} -p ${pass}'
                sh 'docker build --pull --rm -f "Dockerfile" -t ranahesham/springbootapp:v1.2 "."'
                sh 'docker image push ranahesham/springbootapp:v1.2'
                }    
            }                                    
        }
        stage(dev_deployment) {
            steps {
                withKubeConfig([credentialsId: 'mykubeconfig']) {
                    sh 'kubectl delete deployment dev-deployment -n=dev'
                    sh 'kubectl create deployment --image=ranahesham/springbootapp:v1.2 dev-deployment --namespace=dev'
                }
            }
        }
    }
}
```

In Prod Branch with a 2 stages:

Prod Start Stage - Prod Depolyment Stage

```
pipeline {
    agent any
    stages {
        stage(prod_start) {
            steps {
                echo '...................HELLO FROM PROD BRANCH...................'
            }
        }
        stage(prod_deployment) {
            steps {
                withKubeConfig([credentialsId: 'mykubeconfig']) {
                    sh 'kubectl delete deployment prod-deployment -n=prod'
                    sh 'kubectl create deployment --image=ranahesham/springbootapp:v1.2 prod-deployment --namespace=prod'
                }
            }
        }
    }
}
```


**2. Create a multibranch pipelie and add the github repo link**


**3. Download Plugins**

```
Docker Plugin - Git - Github Plugin - SonarQube Scanner - Kubernetes Plugin - kubernetes cli Plugin
```


**4. SonarQube** 

Add sonarqube task to build.gradle file

Run a sonarqube docker image in the azure vm

![image](https://user-images.githubusercontent.com/61191521/204103295-4f733192-6673-4493-b30f-bf5f39d2258b.png)

and open port 9000 in azure 

![open azure port](https://user-images.githubusercontent.com/61191521/204096845-6e5d57b2-3b38-4023-b86c-599132a70db1.jpeg)

then add a token to sonarqube : sonarqube(http://localhost:9000) --> administration --> security --> users --> update tokens

and add jenkins path to sonarqube : sonarqube(http://localhost:9000) --> administration --> configurations --> webhooks --> create

Add sonarqube to sonarqube servers after install its plugin in a configure system and add its token as a secret text and add (azure vm public ip address:9000) as a server url


![image](https://user-images.githubusercontent.com/61191521/204102807-4d1d335f-16fa-43a2-9beb-ddc323455c53.png)


**5. Add Credentials**


for docker registry 

for local minikube at the same vm 


![image](https://user-images.githubusercontent.com/61191521/203888104-e0adafbb-46f5-4f1a-8723-3426434314a5.png)



**6. Add kubernetes as a cloud**


Encrept the data in crt and key files using base64 and replace the pathes of these 3 files with this data in .kube/config file
```
cat /home/rana/.minikube/ca.crt | base64 -w 0; echo
cat /home/rana/.minikube/profiles/minikube/client.crt | base64 -w 0; echo
cat /home/rana/.minikube/profiles/minikube/client.key | base64 -w 0; echo
```
Add the .kube/config file as a secret file 

Add the kubernetes credintials which we created "config (minikube kubeconfig)"

![image](https://user-images.githubusercontent.com/61191521/203982730-edc5719a-081f-4ded-8d05-eca0e9881d44.png)


**7. Start "Scan Multibranch Pipeline Now"**



**NOW THE SPRING BOOT APP HAS BEEN BUILT WITH GRADLE AND DEPLOYED USING DOCKER AND KUBERNETES USING JENKINS**



![image](https://user-images.githubusercontent.com/61191521/203886950-cd2667ad-9867-4138-9fc3-97d3525a071c.png)


![image](https://user-images.githubusercontent.com/61191521/203890915-df57a1ef-3e04-4348-9825-d3e527b6b998.png)


![image](https://user-images.githubusercontent.com/61191521/203887751-900bf004-c6a0-4e10-8089-9df24c2b8a0d.png)



**Dev branch console output**

```
Branch indexing
 > git rev-parse --resolve-git-dir /var/lib/jenkins/caches/git-d5d54b4557093ebc19cf29e017d092f0/.git # timeout=10
Setting origin to https://github.com/rana-hesham/spring-boot.git
 > git config remote.origin.url https://github.com/rana-hesham/spring-boot.git # timeout=10
Fetching origin...
Fetching upstream changes from origin
 > git --version # timeout=10
 > git --version # 'git version 2.25.1'
 > git config --get remote.origin.url # timeout=10
 > git fetch --tags --force --progress -- origin +refs/heads/*:refs/remotes/origin/* # timeout=10
Seen branch in repository origin/dev
Seen branch in repository origin/prod
Seen 2 remote branches
Obtained Jenkinsfile from 3abf6d9ff59dd7ac956842bda9e4d1bc5ce55f66
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/spring-boot-app_dev
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/spring-boot-app_dev/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/rana-hesham/spring-boot.git # timeout=10
Fetching without tags
Fetching upstream changes from https://github.com/rana-hesham/spring-boot.git
 > git --version # timeout=10
 > git --version # 'git version 2.25.1'
 > git fetch --no-tags --force --progress -- https://github.com/rana-hesham/spring-boot.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Checking out Revision 3abf6d9ff59dd7ac956842bda9e4d1bc5ce55f66 (dev)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 3abf6d9ff59dd7ac956842bda9e4d1bc5ce55f66 # timeout=10
Commit message: "Update Jenkinsfile"
 > git rev-list --no-walk 72d156c212bfe57708fdaf995aa73c23da41d359 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (lint)
[Pipeline] echo
...................LINT STAGE................
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (unit_test)
[Pipeline] sh
+ chmod +x gradlew
[Pipeline] sh
+ ./gradlew test
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestJava UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test UP-TO-DATE

BUILD SUCCESSFUL in 10s
4 actionable tasks: 4 up-to-date
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (sonar_qube)
[Pipeline] echo
...................SONARQUBE STAGE................
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (build)
[Pipeline] withCredentials
Masking supported pattern matches of $pass
[Pipeline] {
[Pipeline] sh
+ docker login -u ranahesham -p ****
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /var/lib/jenkins/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[Pipeline] sh
+ docker build --pull --rm -f Dockerfile -t ranahesham/springbootapp:v1.2 .
Sending build context to Docker daemon  789.5kB

Step 1/10 : FROM gradle:7.1.0-jdk11 AS build
7.1.0-jdk11: Pulling from library/gradle
Digest: sha256:6a5c8cb16b5ef665a07949a9e037f34ee25dc6d592997be1a0b58f4a2216cb23
Status: Image is up to date for gradle:7.1.0-jdk11
 ---> 6b88d66d2133
Step 2/10 : RUN mkdir /home/gradle/src
 ---> Using cache
 ---> 4bbffb59d78f
Step 3/10 : COPY --chown=gradle:gradle . /home/gradle/src
 ---> 6d35666dd038
Step 4/10 : WORKDIR /home/gradle/src
 ---> Running in 3f82140ae37d
Removing intermediate container 3f82140ae37d
 ---> 8d8035b187cc
Step 5/10 : RUN gradle build --no-daemon
 ---> Running in 32c40bb42e6d

Welcome to Gradle 7.1!

Here are the highlights of this release:
 - Faster incremental Java compilation
 - Easier source set configuration in the Kotlin DSL

For more details see https://docs.gradle.org/7.1/release-notes.html

To honour the JVM settings for this build a single-use Daemon process will be forked. See https://docs.gradle.org/7.1/userguide/gradle_daemon.html#sec:disabling_the_daemon.
Daemon will be stopped at the end of the build 
> Task :compileJava
> Task :processResources
> Task :classes
> Task :bootJarMainClassName
> Task :bootJar
> Task :jar
> Task :assemble
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> Task :check
> Task :build

BUILD SUCCESSFUL in 1m 14s
7 actionable tasks: 7 executed
Removing intermediate container 32c40bb42e6d
 ---> 54f8be9b4486
Step 6/10 : FROM openjdk:11-jre-slim
11-jre-slim: Pulling from library/openjdk
Digest: sha256:93af7df2308c5141a751c4830e6b6c5717db102b3b31f012ea29d842dc4f2b02
Status: Image is up to date for openjdk:11-jre-slim
 ---> 764a04af3eff
Step 7/10 : EXPOSE 8080
 ---> Using cache
 ---> 2c419b89bba9
Step 8/10 : RUN mkdir /app
 ---> Using cache
 ---> e57d87d770e4
Step 9/10 : COPY --from=build /home/gradle/src/build/libs/demo-0.0.1-SNAPSHOT.jar /app/spring-boot-application.jar
 ---> d3c7f3732ab6
Step 10/10 : ENTRYPOINT ["java", "-XX:+UnlockExperimentalVMOptions", "-Djava.security.egd=file:/dev/./urandom","-jar","/app/spring-boot-application.jar"]
 ---> Running in dbb539c0b06f
Removing intermediate container dbb539c0b06f
 ---> 6131f2594602
Successfully built 6131f2594602
Successfully tagged ranahesham/springbootapp:v1.2
[Pipeline] sh
+ docker image push ranahesham/springbootapp:v1.2
The push refers to repository [docker.io/ranahesham/springbootapp]
a0b4f2f22969: Preparing
747cac21860a: Preparing
d7802b8508af: Preparing
e3abdc2e9252: Preparing
eafe6e032dbd: Preparing
92a4e8a3140f: Preparing
92a4e8a3140f: Waiting
e3abdc2e9252: Layer already exists
eafe6e032dbd: Layer already exists
747cac21860a: Layer already exists
d7802b8508af: Layer already exists
92a4e8a3140f: Layer already exists
a0b4f2f22969: Pushed
v1.2: digest: sha256:bc6ef1cba404f96022dc553f02442111186e74a02abc9484cdc0879b9349ac5b size: 1576
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (dev_deployment)
[Pipeline] withKubeConfig
[Pipeline] {
[Pipeline] sh
+ kubectl delete deployments --all
No resources found
[Pipeline] sh
+ kubectl create deployment --image=ranahesham/springbootapp:v1.2 dev-deployment --namespace=dev
deployment.apps/dev-deployment created
[Pipeline] }
[kubernetes-cli] kubectl configuration cleaned up
[Pipeline] // withKubeConfig
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

**Prod branch console output**

```
Branch indexing
 > git rev-parse --resolve-git-dir /var/lib/jenkins/caches/git-d5d54b4557093ebc19cf29e017d092f0/.git # timeout=10
Setting origin to https://github.com/rana-hesham/spring-boot.git
 > git config remote.origin.url https://github.com/rana-hesham/spring-boot.git # timeout=10
Fetching origin...
Fetching upstream changes from origin
 > git --version # timeout=10
 > git --version # 'git version 2.25.1'
 > git config --get remote.origin.url # timeout=10
 > git fetch --tags --force --progress -- origin +refs/heads/*:refs/remotes/origin/* # timeout=10
Seen branch in repository origin/dev
Seen branch in repository origin/prod
Seen 2 remote branches
Obtained Jenkinsfile from 805424b7b6adfe3ca4f2d10fd316135cb813a846
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/spring-boot-app_prod
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/spring-boot-app_prod/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/rana-hesham/spring-boot.git # timeout=10
Fetching without tags
Fetching upstream changes from https://github.com/rana-hesham/spring-boot.git
 > git --version # timeout=10
 > git --version # 'git version 2.25.1'
 > git fetch --no-tags --force --progress -- https://github.com/rana-hesham/spring-boot.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Checking out Revision 805424b7b6adfe3ca4f2d10fd316135cb813a846 (prod)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 805424b7b6adfe3ca4f2d10fd316135cb813a846 # timeout=10
Commit message: "Update Jenkinsfile"
 > git rev-list --no-walk 4f78ac08c56d0b75e085a1921b06730f61d99698 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (prod_start)
[Pipeline] echo
...................HELLO FROM PROD BRANCH...................
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (prod_deployment)
[Pipeline] withKubeConfig
[Pipeline] {
[Pipeline] sh
+ kubectl delete deployments --all
deployment.apps "prod-deployment" deleted
[Pipeline] sh
+ kubectl create deployment --image=ranahesham/springbootapp:v1.2 prod-deployment --namespace=prod
deployment.apps/prod-deployment created
[Pipeline] }
[kubernetes-cli] kubectl configuration cleaned up
[Pipeline] // withKubeConfig
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```
