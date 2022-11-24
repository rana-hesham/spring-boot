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
