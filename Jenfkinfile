pipeline {
    agent any
    environment {
        KUBECONFIG = '/root/.kube/config'
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
                        docker.build('flask-app:blue', '.')
                        docker.build('flask-app:green', '.')
                    }
                }
            }
        }
        stage('Deploy Blue') {
            steps {
                dir('User') {
                    sh 'kubectl apply -f blue-deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                }
            }
        }
        stage('Switch to Green') {
            steps {
                dir('User') {
                    sh 'kubectl apply -f green-deployment.yaml'
                    sh '''
                    kubectl patch service flask-service \
                      -p '{"spec":{"selector":{"app":"flask","version":"green"}}}'
                    '''
                }
            }
        }
        stage('Cleanup Blue') {
            steps {
                dir('User') {
                    sh 'kubectl delete deployment flask-blue'
                }
            }
        }
        stage('Health Check') {
            steps {
                script {
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost", returnStdout: true).trim()
                    if (response != '200') {
                        echo "Health check failed! Rolling back..."
                        dir('User') {
                            sh '''
                            kubectl patch service flask-service \
                              -p '{"spec":{"selector":{"app":"flask","version":"blue"}}}'
                            '''
                        }
                    }
                }
            }
        }
    }
}
