Pipeline {
    stages {
        stage(lint) {
            steps {
                npm install -g npm-groovy-lint
                npm-groovy-lint
            }
        }
        stage(unit-test) {
            steps {
                junit '**/test-results/test/*.xml'
            }
        }
        stage(sonar-qube) {
            steps {
                sh '...................echo SONARQUBE STAGE................'
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
    }
}
