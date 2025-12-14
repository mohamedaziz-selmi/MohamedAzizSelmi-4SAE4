pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-creds'
        IMAGE_NAME             = 'mohamedazizselmi/student-management'
    }

    stages {
        // 1️⃣ Get SonarQube URL dynamically from Minikube
        stage('Set SonarQube URL') {
            steps {
                script {
                    SONAR_URL = sh(
                        script: "minikube service sonarqube-service --url",
                        returnStdout: true
                    ).trim()
                    echo "Using SonarQube URL: ${SONAR_URL}"
                }
            }
        }

        // 2️⃣ Checkout code
        stage('Checkout code') {
            steps {
                echo "Downloading code from Git..."
                git branch: 'main',
                    url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                    credentialsId: 'd53472d1-7c06-4517-893b-219f23f95bc3'
            }
        }

        // 3️⃣ Maven clean & build
        stage('Maven Clean & Build') {
            steps {
                dir('student-management') {
                    echo "Running Maven clean and build..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        // 4️⃣ SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn sonar:sonar \\
                              -Dsonar.projectKey=student-management \\
                              -Dsonar.projectName=student-management \\
                              -Dsonar.host.url=${SONAR_URL} \\
                              -Dsonar.login=\$SONAR_TOKEN \\
                              -DskipTests \\
                              -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        // 5️⃣ Build Docker Image
        stage('Build Docker Image') {
            steps {
                dir('student-management') {
                    echo "Building Docker image..."
                    sh "docker build -t \$IMAGE_NAME:latest ."
                }
            }
        }

        // 6️⃣ Push Docker Image to Docker Hub
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

        // 7️⃣ Deploy to Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_CREDENTIAL', variable: 'KUBECONFIG')]) {
                    echo "Deploying MySQL, Spring Boot, and SonarQube to Kubernetes..."
                    sh 'kubectl config use-context minikube'

                    sh 'kubectl apply -f student-management/k8s/mysql-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/springboot-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/sonarqube-deployment.yaml --validate=false'

                    sh 'kubectl wait --for=condition=ready pod -l app=mysql --timeout=120s'
                    sh 'kubectl wait --for=condition=ready pod -l app=springboot --timeout=120s'
                    sh 'kubectl wait --for=condition=ready pod -l app=sonarqube --timeout=120s'
                }
            }
        }

        // 8️⃣ Done
        stage('Done') {
            steps {
                echo "Pipeline completed successfully!"
            }
        }
    }
}
