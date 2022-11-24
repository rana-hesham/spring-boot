Pipeline {
    agent any
    stages {
         stage(prod-deployment) {
            steps {
                sh 'kubectl create namespace prod'
                sh 'kubectl create deployment --image=ranahesham/springbootapp:v1.1 prod --namespace=prod --replicas=3'
                }
        }
    }
}
