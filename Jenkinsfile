pipeline {
    agent any
    stages {
        stage(lint) {
            steps {
                npm install -g npm-groovy-lint
                npm-groovy-lint
            }
        }
        stage(unit_test) {
            steps {
                echo '...................TEST STAGE................'
            }
        }
        stage(sonar_qube) {
            steps {
                echo '...................SONARQUBE STAGE................'
                }
        }
        stage(build) {
            steps {
                echo '...................BUILD STAGE................'
            }                                    
        }
    }
}
