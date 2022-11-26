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
