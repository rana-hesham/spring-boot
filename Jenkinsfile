pipeline {
    agent any
    stages {
        stage(lint) {
            script {
                try {
                    sh 'chmod +x gradlew'
                    sh './gradlew lint'
                } finally {
                    step([$class: 'ArtifactArchiver', artifacts: 'app/build/reports/staticAnalysis/lint/', fingerprint: true])
                }
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
