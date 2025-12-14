pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mohamedazizselmi/student-management:latest'
        SONAR_URL = 'http://127.0.0.1:41919'  // NodePort URL of SonarQube
    }

    stages {

        stage('Checkout code') {
            steps {
                echo "Downloading code from Git..."
                git branch: 'main', url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git', credentialsId: 'your-git-credentials'
            }
        }

        stage('Maven Clean & Build') {
            steps {
                dir('student-management') {
                    echo "Running Maven clean and build..."
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
                            mvn sonar:sonar \\
                              -Dsonar.projectKey=student-management \\
                              -Dsonar.projectName=student-management \\
                              -Dsonar.host.url=$SONAR_URL \\
                              -Dsonar.login=\$SONAR_TOKEN \\
                              -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker build -t $DOCKER_IMAGE student-management
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl config use-context minikube
                        kubectl apply -f student-management/k8s/mysql-deployment.yaml --validate=false
                        kubectl apply -f student-management/k8s/student-management-deployment.yaml --validate=false
                        kubectl apply -f student-management/k8s/student-management-service.yaml --validate=false
                    """
                }
            }
        }

        stage('Done') {
            steps {
                echo "Pipeline finished successfully!"
            }
        }
    }
}
