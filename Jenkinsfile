pipeline {
    agent any
    environment {
        // Set Windows-style kubeconfig path
        KUBECONFIG = 'C:\\Users\\jenkins\\.kube\\config'
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
                        // Use Windows docker.exe commands
                        bat 'docker build -t flask-app:blue .'
                        bat 'docker build -t flask-app:green .'
                    }
                }
            }
        }
        stage('Deploy Blue') {
            steps {
                dir('User') {
                    // Use kubectl.exe with Windows batch
                    bat 'kubectl apply -f blue-deployment.yaml'
                    bat 'kubectl apply -f service.yaml'
                }
            }
        }
        stage('Switch to Green') {
            steps {
                dir('User') {
                    bat 'kubectl apply -f green-deployment.yaml'
                    bat """
                    kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"green\\"}}}"
                    """
                }
            }
        }
        stage('Cleanup Blue') {
            steps {
                dir('User') {
                    bat 'kubectl delete deployment flask-blue'
                }
            }
        }
        stage('Health Check') {
            steps {
                script {
                    def response = bat(script: 'curl -s -o NUL -w "%{http_code}" http://localhost', returnStdout: true).trim()
                    if (response != '200') {
                        echo "Health check failed! Rolling back..."
                        dir('User') {
                            bat """
                            kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"blue\\"}}}"
                            """
                        }
                    }
                }
            }
        }
    }
}
