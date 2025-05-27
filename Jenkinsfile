pipeline {
    agent any

    environment {
        // Make sure this path is accessible to Jenkins
        KUBECONFIG = 'C:/Program Files/Jenkins/.kube/config' // Replace with actual path if needed
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Blue and Green Images') {
            steps {
                dir('User') {
                    script {
                        bat 'docker build -t flask-app:blue .'
                        bat 'docker build -t flask-app:green .'
                    }
                }
            }
        }

        stage('Deploy Blue') {
            steps {
                dir('User\\k8s') {
                    bat 'kubectl apply -f blue-deployment.yaml'
                    bat 'kubectl apply -f service.yaml'
                }
            }
        }

        stage('Deploy Green and Switch Traffic') {
            steps {
                dir('User\\k8s') {
                    bat 'kubectl apply -f green-deployment.yaml'
                    // Correctly escaped JSON string for Windows shell
                    bat '''kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"green\\"}}}"'''
                }
            }
        }

        stage('Cleanup Blue') {
            steps {
                dir('User\\k8s') {
                    bat 'kubectl delete deployment flask-blue'
                }
            }
        }
    }

    post {
        failure {
            echo 'Deployment failed. Check the logs for more information.'
        }
    }
}
