pipeline {
    agent any

    environment {
        KUBECONFIG = 'C:/Program Files/Jenkins/.kube/config'
        DOCKERHUB_USER = 'swathikulk'   // Replace with your Docker Hub username
        IMAGE_TAG = "${env.BUILD_NUMBER}"            // Unique tag for each build
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
                        bat "docker build -t %DOCKERHUB_USER%/flask-app:blue-%IMAGE_TAG% ."
                        bat "docker tag %DOCKERHUB_USER%/flask-app:blue-%IMAGE_TAG% %DOCKERHUB_USER%/flask-app:green-%IMAGE_TAG%"
                    }
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                        bat "docker push %DOCKERHUB_USER%/flask-app:blue-%IMAGE_TAG%"
                        bat "docker push %DOCKERHUB_USER%/flask-app:green-%IMAGE_TAG%"
                    }
                }
            }
        }

        stage('Deploy Blue') {
            steps {
                dir('User\\k8s') {
                    script {
                        bat """
                        kubectl apply -f blue-deployment.yaml
                        kubectl apply -f service.yaml
                        """
                    }
                }
            }
        }

        stage('Deploy Green and Switch Traffic') {
            steps {
                dir('User\\k8s') {
                    script {
                        bat 'kubectl apply -f green-deployment.yaml'
                        bat '''kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"green\\"}}}"'''
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

    post {
        failure {
            echo 'Deployment failed. Check the logs for more information.'
        }
    }
}
