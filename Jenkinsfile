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
                dir('student-management') {  // Changed from student-management/student-management
                    echo "Cleaning and building Maven project..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {  // Changed here too
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
                dir('student-management') {  // Changed here too
                    echo "Building Docker image..."
                    sh 'docker build -t ${IMAGE_NAME}:latest .'
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                dir('student-management') {  // Changed here too
                    echo "Logging in and pushing to Docker Hub..."
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh 'echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin'
                        sh 'docker push ${IMAGE_NAME}:latest'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                dir('student-management/k8s') {  // Changed here too
                    withEnv(['KUBECONFIG=/var/lib/jenkins/.kube/config']) {
                        sh 'kubectl config use-context minikube'
                        sh 'kubectl apply -f mysql-deployment.yaml --validate=false'
                        sh 'kubectl apply -f springboot-deployment.yaml --validate=false'
                        sh 'kubectl wait --for=condition=ready pod -l app=mysql --timeout=120s'
                        sh 'kubectl wait --for=condition=ready pod -l app=springboot --timeout=120s'
                    }
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
