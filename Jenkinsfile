pipeline {
    agent any
    environment {
        // Adjust the path to kubeconfig if needed
        KUBECONFIG = 'C:/Program Files/Jenkins/.kube/config'
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

        stage('Deploy and Switch to Green') {
            steps {
                dir('User\\k8s') {
                    bat 'kubectl apply -f green-deployment.yaml'
                    bat 'kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"green\\"}}}"'
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

        stage('Start Minikube') {
            steps {
                bat '"C:\\Program Files\\Kubernetes\\Minikube\\minikube.exe" start'
            }
        }

        stage('Expose URL') {
            steps {
                script {
                    def url = bat(returnStdout: true, script: '"C:\\Program Files\\Kubernetes\\Minikube\\minikube.exe" service flask-service --url').trim()
                    echo "Service URL: ${url}"
                }
            }
        }


    }
}
