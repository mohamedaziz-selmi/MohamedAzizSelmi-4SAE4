pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'the credentials for docker hub'
        IMAGE_NAME = 'mohamedazizselmi/student-management'
    }

    stages {
        stage('Checkout code') {
            steps {
                echo "Téléchargement du code..."
                git(
                    branch: 'main',
                    url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                    credentialsId: 'd53472d1-7c06-4517-893b-219f23f95bc3'
                )
            }
        }

        stage('Maven Clean') {
            steps {
                dir('student-management') {
                    echo "Nettoyage du projet Maven..."
                    sh "mvn clean -DskipTests"
                }
            }
        }

        stage('Maven Package') {
            steps {
                dir('student-management') {
                    echo "Build Maven (package)..."
                    sh "mvn install -DskipTests"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('student-management') {
                    echo "Construction de l’image Docker..."
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management') {
                    echo "Connexion et push vers Docker Hub..."
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_HUB_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
    steps {
        withCredentials([file(credentialsId: 'KUBECONFIG_CREDENTIAL', variable: 'KUBECONFIG')]) {
            sh 'kubectl apply -f student-management/k8s/mysql-deployment.yaml'
            sh 'kubectl apply -f student-management/k8s/springboot-deployment.yaml'
        }
    }
}


        stage('Done') {
            steps {
                echo "Pipeline completed"
            }
        }
    }
}
