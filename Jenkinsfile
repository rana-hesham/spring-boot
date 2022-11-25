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
                    sh 'kubectl create -f prod_deployment.yaml'
                }
            }
        }
    }
}
