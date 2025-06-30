// pipeline {
//     agent any

//     environment {
//         KUBECONFIG = 'C:/Users/Swathi Kulkarni/.kube/config'
//         DOCKERHUB_USER = 'supriya334'
//         IMAGE_TAG = "${env.BUILD_NUMBER}"
//     }

//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Build Blue and Green Images') {
//             steps {
//                 dir('User') {
//                     script {
//                         echo "Building Docker images with tag: ${env.IMAGE_TAG}"
//                         bat "docker build -t %DOCKERHUB_USER%/flask-app:blue-%IMAGE_TAG% ."
//                         bat "docker tag %DOCKERHUB_USER%/flask-app:blue-%IMAGE_TAG% %DOCKERHUB_USER%/flask-app:green-%IMAGE_TAG%"
//                     }
//                 }
//             }
//         }

//         stage('Push Images to DockerHub') {
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
//                     script {
//                         echo "Logging in to DockerHub as ${env.DOCKERHUB_USER}"
//                         bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
//                         bat "docker push %DOCKERHUB_USER%/flask-app:blue-%IMAGE_TAG%"
//                         bat "docker push %DOCKERHUB_USER%/flask-app:green-%IMAGE_TAG%"
//                     }
//                 }
//             }
//         }

//         stage('Deploy Blue') {
//             steps {
//                 dir('User\\k8s') {
//                     script {
//                         echo "Deploying Blue version with tag: ${env.IMAGE_TAG}"
//                         bat "powershell -Command \"(Get-Content blue-deployment.yaml) -replace '__BUILD_NUMBER__', '${env.IMAGE_TAG}' | Set-Content blue-deployment.yaml\""
//                         bat 'kubectl apply -f blue-deployment.yaml'
//                         bat 'kubectl apply -f service.yaml'
//                     }
//                 }
//             }
//         }

//         stage('Deploy Green and Switch Traffic') {
//             steps {
//                 dir('User\\k8s') {
//                     script {
//                         echo "Deploying Green version and switching traffic"
//                         bat "powershell -Command \"(Get-Content green-deployment.yaml) -replace '__BUILD_NUMBER__', '${env.IMAGE_TAG}' | Set-Content green-deployment.yaml\""
//                         bat 'kubectl apply -f green-deployment.yaml'
//                         bat 'kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"green\\"}}}"'
//                     }
//                 }
//             }
//         }

//         stage('Cleanup Blue') {
//             steps {
//                 dir('User\\k8s') {
//                     script {
//                         echo "Deleting blue deployment"
//                         bat 'kubectl delete deployment flask-blue || echo "No existing blue deployment found."'
//                     }
//                 }
//             }
//         }
//     }

//     post {
//         failure {
//             echo 'Deployment failed. Check the logs for more information.'
//         }
//     }
// }

pipeline {
    agent any
    environment {
        DOCKER_USER = 'supriya334'
        DOCKER_IMAGE = 'flask-app'
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
                        echo "Building Docker images with tag: ${BUILD_NUMBER}"
                        bat "docker build -t ${DOCKER_USER}/${DOCKER_IMAGE}:blue-${BUILD_NUMBER} ."
                        bat "docker tag ${DOCKER_USER}/${DOCKER_IMAGE}:blue-${BUILD_NUMBER} ${DOCKER_USER}/${DOCKER_IMAGE}:green-${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        echo "Logging in to DockerHub"
                        bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                        bat "docker push ${DOCKER_USER}/${DOCKER_IMAGE}:blue-${BUILD_NUMBER}"
                        bat "docker push ${DOCKER_USER}/${DOCKER_IMAGE}:green-${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy Blue') {
            steps {
                dir('User/k8s') {
                    script {
                        echo "Deploying Blue version: ${BUILD_NUMBER}"
                        bat "powershell -Command \"(Get-Content blue-deployment.yaml) -replace 'BUILD_NUMBER', '${BUILD_NUMBER}' | Set-Content blue-deployment.yaml\""
                        bat "kubectl apply -f blue-deployment.yaml"
                        bat "kubectl apply -f service.yaml"
                    }
                }
            }
        }

        stage('Deploy Green') {
            steps {
                dir('User/k8s') {
                    script {
                        echo "Deploying Green version: ${BUILD_NUMBER}"
                        bat "powershell -Command \"(Get-Content green-deployment.yaml) -replace 'BUILD_NUMBER', '${BUILD_NUMBER}' | Set-Content green-deployment.yaml\""
                        bat "kubectl apply -f green-deployment.yaml"
                    }
                }
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    def userInput = input(
                        message: 'Switch traffic to Green?',
                        parameters: [
                            choice(name: 'Approve', choices: ['Yes', 'No'], description: 'Approve deployment to Green?')
                        ]
                    )
                    if (userInput == 'Yes') {
                        echo '✅ Switching traffic to Green'
                        bat 'kubectl patch service flask-service -p "{\"spec\":{\"selector\":{\"app\":\"flask\",\"version\":\"green\"}}}"'
                    } else {
                        error("❌ Deployment not approved. Halting pipeline.")
                    }
                }
            }
        }

        stage('Cleanup Blue') {
            steps {
                echo "🧹 You can add blue cleanup logic here if needed"
            }
        }
    }

    post {
        failure {
            echo '🚨 Deployment failed. Check logs.'
        }
        success {
            echo '🎉 Deployment succeeded!'
        }
    }
}