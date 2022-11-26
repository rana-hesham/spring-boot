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
