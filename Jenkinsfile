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
                echo '...................BUILD STAGE................'
            }                                    
        }
    }
}
