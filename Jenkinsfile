pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-creds'
        IMAGE_NAME             = 'mohamedazizselmi/student-management'
        // Use Minikube IP instead of localhost if Jenkins runs in Docker/VM
        MINIKUBE_IP            = sh(script: "minikube ip", returnStdout: true).trim()
        SONAR_URL              = "http://${MINIKUBE_IP}:9000"
    }

    stages {
        // 1️⃣ Checkout code
        stage('Checkout code') {
            steps {
                echo "Downloading code from Git..."
                git branch: 'main',
                    url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                    credentialsId: 'd53472d1-7c06-4517-893b-219f23f95bc3'
            }
        }

        // 2️⃣ Maven clean & build
        stage('Maven Clean & Build') {
            steps {
                dir('student-management') {
                    echo "Running Maven clean and build..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        // 3️⃣ SonarQube Analysis
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
                              -DskipTests \\
                              -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        // 4️⃣ Build Docker image
        stage('Build Docker Image') {
            steps {
                dir('student-management') {
                    echo "Building Docker image..."
                    sh "docker build -t \$IMAGE_NAME:latest ."
                }
            }
        }

        // 5️⃣ Push Docker image to Docker Hub
        stage('Push Docker Image') {
            steps {
                dir('student-management') {
                    echo "Logging into Docker Hub and pushing image..."
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker push \$IMAGE_NAME:latest
                        """
                    }
                }
            }
        }

        // 6️⃣ Deploy to Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_CREDENTIAL', variable: 'KUBECONFIG')]) {
                    echo "Deploying MySQL, Spring Boot, and SonarQube to Kubernetes..."
                    
                    // Use KUBECONFIG from credentials
                    sh 'export KUBECONFIG=$KUBECONFIG'
                    
                    // Apply Kubernetes YAMLs
                    sh 'kubectl apply -f student-management/k8s/mysql-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/springboot-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/sonarqube-deployment.yaml --validate=false'

                    // Wait for pods to be ready
                    sh 'kubectl wait --for=condition=ready pod -l app=mysql --timeout=180s'
                    sh 'kubectl wait --for=condition=ready pod -l app=springboot --timeout=180s'
                    sh 'kubectl wait --for=condition=ready pod -l app=sonarqube --timeout=180s'
                }
            }
        }

        // 7️⃣ Done
        stage('Done') {
            steps {
                echo "Pipeline completed successfully!"
            }
        }
    }
}

