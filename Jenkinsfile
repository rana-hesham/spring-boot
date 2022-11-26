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
                sh 'chmod +x gradlew'
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
