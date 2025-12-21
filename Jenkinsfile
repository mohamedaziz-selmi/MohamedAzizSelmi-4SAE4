pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_URL = 'http://localhost:9000'
    }
    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "📦 Cloning repository from GitHub..."
                    sh 'rm -rf student-management'
                    
                    // Use HTTPS with token
                    git url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                        branch: 'main',
                        credentialsId: 'github-https-token'
                        
                    echo "✅ Repository cloned successfully"
                }
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
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        echo "Running SonarQube analysis..."
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.projectName=student-management \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -DskipTests \
                            -Dsonar.java.binaries=target/classes
                        '''
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                dir('student-management') {
                    echo "Building Docker image..."
                    sh 'docker build -t ${IMAGE_NAME}:latest .'
                }
            }
        }
        stage('Push Docker Image') {
    steps {
        dir('student-management') {
            echo "Pushing to Docker Hub..."
            withCredentials([usernamePassword(
                credentialsId: 'docker-hub-creds',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                script {
                    try {
                        timeout(time: 2, unit: 'MINUTES') {
                            sh 'echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin'
                            sh 'docker push ${IMAGE_NAME}:latest'
                            echo "✅ Pushed to Docker Hub"
                        }
                    } catch (Exception e) {
                        echo "⚠️ Docker Hub push failed, loading into Minikube instead..."
                        sh 'minikube image load ${IMAGE_NAME}:latest'
                        echo "✅ Image loaded into Minikube"
                    }
                }
            }
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                dir('student-management/k8s') {
                    withEnv(['KUBECONFIG=/var/lib/jenkins/.kube/config']) {
                        sh 'kubectl config use-context minikube'
                        
                        // Apply or create resources (use create or replace for immutable resources)
                        sh '''
                            # Apply secret (can be updated)
                            kubectl apply -f mysql-secret.yaml
                            
                            # PVC - only create if doesn't exist (immutable)
                            kubectl get pvc mysql-pvc || kubectl create -f mysql-pvc.yaml
                            
                            # Apply MySQL deployment and service
                            kubectl apply -f mysql-deployment.yaml
                            kubectl apply -f mysql-service.yaml
                            
                            # Apply SpringBoot deployment and service
                            kubectl apply -f springboot-deployment.yaml
                            kubectl apply -f springboot-service.yaml
                            
                            # Wait for pods to be ready
                            echo "Waiting for MySQL to be ready..."
                            kubectl wait --for=condition=ready pod -l app=mysql --timeout=180s || true
                            
                            echo "Waiting for SpringBoot to be ready..."
                            kubectl wait --for=condition=ready pod -l app=springboot --timeout=300s || true
                            
                            # Show deployment status
                            kubectl get pods -l app=mysql
                            kubectl get pods -l app=springboot
                        '''
                    }
                }
            }
        }
        stage('Done') {
            steps {
                echo "✅ Pipeline completed successfully!"
                echo "🚀 Application deployed to Kubernetes"
            }
        }
    }
    post {
        success {
            echo "✅ Build and deployment completed successfully!"
        }
        failure {
            echo "❌ Build or deployment failed. Check the logs above."
        }
        always {
            echo "🧹 Cleaning up workspace..."
            cleanWs()
        }
    }
}
