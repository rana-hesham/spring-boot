pipeline {
    agent any
    stages {
        stage(lint) {
            steps {
                echo '...................LINT STAGE................'
            }
        }
        stage(unit_test) {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew test'
            }
        }
        stage(sonar_qube) {
            steps {
                echo '...................SONARQUBE STAGE................'
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
                sh 'echo Docker@112022 | sudo -S kubectl create deployment --image=ranahesham/springbootapp:v1.2 dev-from-jenkins --namespace=dev --replicas=3'
            }
        }
    }
}
