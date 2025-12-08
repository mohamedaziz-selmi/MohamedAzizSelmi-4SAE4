pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'the credentials for docker hub'
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_TOKEN = credentials('sonar-token')  // keep this ID, you need to create it in Jenkins as secret text
        SONAR_URL = 'http://192.168.49.2:31001'      // replace with your local SonarQube URL if different
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

        stage('Maven Build') {
            steps {
                dir('student-management') {
                    echo "Build Maven (install)..."
                    sh "mvn install -DskipTests"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    echo "Analyse SonarQube..."
                    mvn sonar:sonar -Dsonar.projectKey=student-management -Dsonar.projectName=student-management -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.token=$SONAR_TOKEN -DskipTests -Dsonar.java.binaries=target/classes

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
                    echo "Déploiement sur Kubernetes..."
                    sh 'kubectl apply -f student-management/k8s/mysql-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/springboot-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/sonarqube-deployment.yaml --validate=false'
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
