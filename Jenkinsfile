
pipeline {
    agent any

    stages {
        stage('Checkout from GitHub') {
            steps {
                echo 'Pipeline loaded from the GitHub repository.'
                sh 'pwd && ls -la'
            }
        }

        stage('Build Green Image') {
            steps {
                sh '''
                    cd /opt/vle8-bluegreen
                    eval "$(sudo -u ubuntu minikube docker-env)"
                    docker build -t myapp:v2 ./green
                '''
            }
        }

        stage('Deploy Green') {
            steps {
                sh '''
                    kubectl --kubeconfig=/var/lib/jenkins/.kube/config \
                        apply -f /opt/vle8-bluegreen/deployment-green.yaml

                    kubectl --kubeconfig=/var/lib/jenkins/.kube/config \
                        rollout status deployment/myapp-green --timeout=120s
                '''
            }
        }

        stage('Approve Traffic Switch') {
            steps {
                input message: 'Green is deployed. Switch live traffic from Blue to Green?',
                      ok: 'Switch to Green'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''
                    kubectl --kubeconfig=/var/lib/jenkins/.kube/config \
                        patch service myapp-service \
                        -p '{"spec":{"selector":{"app":"myapp","color":"green"}}}'

                    kubectl --kubeconfig=/var/lib/jenkins/.kube/config \
                        get service myapp-service
                '''
            }
        }
    }

    post {
        success {
            echo 'GitHub-based blue-green pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
