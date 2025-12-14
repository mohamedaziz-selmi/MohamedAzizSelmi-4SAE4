pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_URL = 'http://192.168.49.2:31666'
    }
    stages {

        stage('Checkout code via SSH') {
            steps {
                echo "Cleaning old folder and cloning repository via SSH..."
                sh 'rm -rf student-management'
                sh 'git clone -b main git@github.com:fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git student-management'
            }
        }

        stage('Maven Clean & Build') {
            steps {
                dir('student-management/student-management') {
                    echo "Cleaning and building Maven project..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

      stage('SonarQube Analysis') {
    steps {
        dir('student-management/student-management') {
            withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                echo "Running SonarQube analysis..."
                sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=student-management \
                    -Dsonar.projectName=student-management \
                    -Dsonar.host.url=http://192.168.49.2:31666 \
                    -Dsonar.login=$SONAR_TOKEN \
                    -DskipTests \
                    -Dsonar.java.binaries=target/classes
                """
            }
        }
    }
}


        stage('Build Docker Image') {
            steps {
                dir('student-management/student-management') {
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
                dir('student-management/student-management/k8s') {
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
