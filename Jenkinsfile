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
        KUBECONFIG = 'C:/Users/Swathi Kulkarni/.kube/config'
        DOCKERHUB_USER = 'supriya334'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        PREVIOUS_BUILD = "${BUILD_NUMBER.toInteger() - 1}"
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
                        echo "Building Docker images with tag: ${env.IMAGE_TAG}"
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
                        echo "Logging in to DockerHub as ${env.DOCKERHUB_USER}"
                        bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
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
                        echo "Deploying Blue version with tag: ${env.IMAGE_TAG}"
                        bat "powershell -Command \"(Get-Content blue-deployment.yaml) -replace '_BUILD_NUMBER_', '${env.IMAGE_TAG}' | Set-Content blue-deployment.yaml\""
                        bat 'kubectl apply -f blue-deployment.yaml'
                        bat 'kubectl apply -f service.yaml'
                    }
                }
            }
        }

        stage('Deploy Green and Switch Traffic') {
            steps {
                dir('User\\k8s') {
                    script {
                        echo "Deploying Green version and switching traffic"
                        bat "powershell -Command \"(Get-Content green-deployment.yaml) -replace '_BUILD_NUMBER_', '${env.IMAGE_TAG}' | Set-Content green-deployment.yaml\""
                        bat 'kubectl apply -f green-deployment.yaml'
                        bat 'kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"green\\"}}}"'
                    }
                }
            }
        }

        stage('User Approval for Green Deployment') {
            steps {
                script {
                    def userInput = input(
                        id: 'userInput', message: 'Is the GREEN deployment healthy?', parameters: [
                            choice(name: 'Confirm', choices: 'Yes\nNo', description: 'Choose Yes to proceed or No to rollback to previous Blue')
                        ]
                    )

                    if (userInput == 'No') {
                        echo "User rejected Green. Rolling back to previous Blue deployment."

                        dir('User\\k8s') {
                            bat "powershell -Command \"(Get-Content blue-deployment.yaml) -replace '__BUILD_NUMBER__', '${env.PREVIOUS_BUILD}' | Set-Content blue-deployment.yaml\""
                            bat 'kubectl apply -f blue-deployment.yaml'
                            bat 'kubectl patch service flask-service -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"flask\\",\\"version\\":\\"blue\\"}}}"'
                        }

                        // Optionally delete green if rollback is triggered
                        dir('User\\k8s') {
                            bat 'kubectl delete deployment flask-green || echo "No existing green deployment found."'
                        }
                    }
                }
            }
        }

        stage('Cleanup Blue') {
            when {
                expression {
                    // Only run if user approved the Green deployment
                    return currentBuild.rawBuild.getActions(hudson.model.ParametersAction.class)
                        .find { action -> action.getParameter('Confirm')?.value == 'Yes' } != null
                }
            }
            steps {
                dir('User\\k8s') {
                    script {
                        echo "Deleting blue deployment"
                        bat 'kubectl delete deployment flask-blue || echo "No existing blue deployment found."'
                    }
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