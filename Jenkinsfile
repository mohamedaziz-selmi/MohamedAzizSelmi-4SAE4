pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_URL = 'http://192.168.49.2:31666'
    }
    stages {
        stage('Checkout code') {
            steps {
                echo "Downloading code..."
                git branch: 'main',
                    url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                    credentialsId: 'd53472d1-7c06-4517-893b-219f23f95bc3'
            }
        }

        stage('Maven Clean & Build') {
            steps {
                dir('student-management') {
                    echo "Cleaning and building Maven project..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    echo "Running SonarQube analysis..."
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.projectName=student-management \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.login=\$SONAR_TOKEN \
                            -DskipTests \
                            -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('student-management') {
                    echo "Building Docker image..."
                    sh "docker build -t \$IMAGE_NAME:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management') {
                    echo "Logging in and pushing to Docker Hub..."
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker push \$IMAGE_NAME:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes via WSL..."
                dir('student-management/k8s') {
                    // Call kubectl inside WSL
                    sh 'wsl kubectl config use-context minikube'
                    sh 'wsl kubectl apply -f mysql-deployment.yaml --validate=false'
                    sh 'wsl kubectl apply -f springboot-deployment.yaml --validate=false'
                    sh 'wsl kubectl apply -f sonarqube-deployment.yaml --validate=false'
                    sh 'wsl kubectl wait --for=condition=ready pod -l app=mysql --timeout=120s'
                    sh 'wsl kubectl wait --for=condition=ready pod -l app=springboot --timeout=120s'
                    sh 'wsl kubectl wait --for=condition=ready pod -l app=sonarqube --timeout=120s'
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
