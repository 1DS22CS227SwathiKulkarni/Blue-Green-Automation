pipeline {
    agent any
    environment {
        // Set Windows-style kubeconfig path (adjust username as needed)
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

        stage('Health Check and Rollback if Needed') {
            steps {
                script {
                    echo "Running health check on green deployment..."
                    def response = bat(script: 'curl -s -o NUL -w "%{http_code}" http://localhost:30000', returnStdout: true).trim()

                    if (response != '200') {
                        echo "❌ Health check failed! Rolling back to Blue..."

                        dir('User\\k8s') {
                            bat 'kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"blue\\"}}}"'
                            bat 'kubectl delete deployment flask-green'
                        }

                        error("Green deployment failed. Rolled back to blue.")
                    } else {
                        echo "✅ Health check passed. Proceeding to clean up blue deployment..."
                    }
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
}
