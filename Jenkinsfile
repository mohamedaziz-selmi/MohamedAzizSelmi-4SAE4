pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-creds'   // Replace with your Docker Hub credential ID
        SONAR_TOKEN = credentials('sonar-token')      // Jenkins secret text with your SonarQube token
        SONAR_URL = 'http://127.0.0.1:44209'          // Replace with your SonarQube URL
        IMAGE_NAME = 'mohamedazizselmi/student-management'
    }

    stages {
        stage('Checkout code') {
            steps {
                echo "Checking out code..."
                git(
                    branch: 'main',
                    url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                    credentialsId: 'd53472d1-7c06-4517-893b-219f23f95bc3'
                )
            }
        }

        stage('Maven Build') {
            steps {
                dir('student-management') {
                    echo "Building Maven project..."
                    sh "mvn clean install -DskipTests"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    echo "Running SonarQube analysis..."
                    sh "mvn sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.projectName='student-management' \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.token=${SONAR_TOKEN} \
                        -DskipTests \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('student-management') {
                    echo "Building Docker image..."
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management') {
                    echo "Pushing Docker image to Docker Hub..."
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
                    echo "Deploying to Kubernetes..."
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
