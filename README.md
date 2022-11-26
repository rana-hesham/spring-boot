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

In Prod Branch with an 1 stage:

Prod Depolyment Stage

```
pipeline {
    agent any
    stages {
        stage(prod_deployment) {
            steps {
                withKubeConfig([credentialsId: 'mykubeconfig']) {
                    echo '...................HELLO FROM PROD BRANCH...................'
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
Obtained Jenkinsfile from 81536cb246dcbeb29e8af29179fe91475d3fd4c1
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
Checking out Revision 81536cb246dcbeb29e8af29179fe91475d3fd4c1 (dev)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 81536cb246dcbeb29e8af29179fe91475d3fd4c1 # timeout=10
Commit message: "Update README.md"
 > git rev-list --no-walk bb70f33552a7dcce955cca9e7224617eb0a7f288 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (lint)
[Pipeline] sh
+ chmod +x gradlew
[Pipeline] sh
+ ./gradlew lint
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestJava UP-TO-DATE
> Task :lintGradle
> Task :autoLintGradle

BUILD SUCCESSFUL in 12s
5 actionable tasks: 2 executed, 3 up-to-date
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (unit_test)
[Pipeline] sh
+ ./gradlew test
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestJava UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test UP-TO-DATE
> Task :autoLintGradle

BUILD SUCCESSFUL in 1s
5 actionable tasks: 1 executed, 4 up-to-date
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (sonar_qube)
[Pipeline] withSonarQubeEnv
Injecting SonarQube environment variables using the configuration: SonarQube
[Pipeline] {
[Pipeline] sh
+ ./gradlew sonarqube
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestJava UP-TO-DATE
> Task :sonarqube
> Task :autoLintGradle

BUILD SUCCESSFUL in 9s
5 actionable tasks: 2 executed, 3 up-to-date
[Pipeline] }
[Pipeline] // withSonarQubeEnv
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
Sending build context to Docker daemon  1.097MB

Step 1/10 : FROM gradle:7.1.0-jdk11 AS build
7.1.0-jdk11: Pulling from library/gradle
Digest: sha256:6a5c8cb16b5ef665a07949a9e037f34ee25dc6d592997be1a0b58f4a2216cb23
Status: Image is up to date for gradle:7.1.0-jdk11
 ---> 6b88d66d2133
Step 2/10 : RUN mkdir /home/gradle/src
 ---> Using cache
 ---> 4bbffb59d78f
Step 3/10 : COPY --chown=gradle:gradle . /home/gradle/src
 ---> f19a10b875c7
Step 4/10 : WORKDIR /home/gradle/src
 ---> Running in 887a2c80891e
Removing intermediate container 887a2c80891e
 ---> 785d799b37f8
Step 5/10 : RUN gradle build --no-daemon
 ---> Running in 6d662aa89f43

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
> Task :autoLintGradle

BUILD SUCCESSFUL in 1m 25s
8 actionable tasks: 8 executed
Removing intermediate container 6d662aa89f43
 ---> 04a38997a7de
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
 ---> 67f27f5aded2
Step 10/10 : ENTRYPOINT ["java", "-XX:+UnlockExperimentalVMOptions", "-Djava.security.egd=file:/dev/./urandom","-jar","/app/spring-boot-application.jar"]
 ---> Running in b0280f3c52de
Removing intermediate container b0280f3c52de
 ---> c194c79f1ede
Successfully built c194c79f1ede
Successfully tagged ranahesham/springbootapp:v1.2
[Pipeline] sh
+ docker image push ranahesham/springbootapp:v1.2
The push refers to repository [docker.io/ranahesham/springbootapp]
4b0253545330: Preparing
747cac21860a: Preparing
d7802b8508af: Preparing
e3abdc2e9252: Preparing
eafe6e032dbd: Preparing
92a4e8a3140f: Preparing
92a4e8a3140f: Waiting
e3abdc2e9252: Layer already exists
747cac21860a: Layer already exists
d7802b8508af: Layer already exists
eafe6e032dbd: Layer already exists
92a4e8a3140f: Layer already exists
4b0253545330: Pushed
v1.2: digest: sha256:4a8f0fb48270963e7abf33e3ea90b425cf4249bb174238e5cbb6896683d2e04f size: 1576
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (dev_deployment)
[Pipeline] withKubeConfig
[Pipeline] {
[Pipeline] sh
+ kubectl delete deployment dev-deployment -n=dev
deployment.apps "dev-deployment" deleted
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
Obtained Jenkinsfile from 3812058b944872f7929406b367fba018eedecfa2
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
Checking out Revision 3812058b944872f7929406b367fba018eedecfa2 (prod)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 3812058b944872f7929406b367fba018eedecfa2 # timeout=10
Commit message: "Update Jenkinsfile"
 > git rev-list --no-walk 3812058b944872f7929406b367fba018eedecfa2 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (prod_deployment)
[Pipeline] withKubeConfig
[Pipeline] {
[Pipeline] echo
...................HELLO FROM PROD BRANCH...................
[Pipeline] sh
+ kubectl delete deployment prod-deployment -n=prod
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
