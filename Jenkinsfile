pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-creds'       // Your Jenkins Docker Hub credential ID
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_HOST_URL = 'http://127.0.0.1:44209'         // SonarQube URL reachable from Jenkins
        SONAR_TOKEN = credentials('sonar-token')          // Jenkins credential storing your token
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

        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    echo "Exécution de l'analyse SonarQube..."
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.projectName='student-management' \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.token=${SONAR_TOKEN} \
                        -DskipTests
                        -Dsonar.java.binaries=target/classes"

                
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
                    sh 'kubectl apply -f student-management/k8s/sonarqube-deployment.yaml'
                }
            }
        }

        stage('Done') {
            steps {
                echo "Pipeline completed successfully!"
            }
        }
    }
}
