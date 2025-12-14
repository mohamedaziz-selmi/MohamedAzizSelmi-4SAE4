pipeline {
    agent any

    environment {
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_PROJECT_KEY = 'student-management'
        SONAR_PROJECT_NAME = 'student-management'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Downloading code from GitHub..."
                git branch: 'main',
                    url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                    credentialsId: 'GITHUB_CREDENTIAL_ID' // your GitHub token
            }
        }

        stage('Maven Build') {
            steps {
                dir('student-management') {
                    echo "Building Maven project..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    echo "Running SonarQube analysis..."
                    withCredentials([string(credentialsId: 'SONAR_TOKEN_CREDENTIAL', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn sonar:sonar \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                              -Dsonar.host.url=http://127.0.0.1:40995 \
                              -Dsonar.token=\$SONAR_TOKEN \
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
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management') {
                    echo "Pushing Docker image to Docker Hub..."
                    withCredentials([usernamePassword(
                        credentialsId: 'DOCKER_HUB_CREDS', 
                        usernameVariable: 'DOCKER_USER', 
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker push ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_CREDENTIAL', variable: 'KUBECONFIG')]) {
                    echo "Deploying MySQL, Spring Boot, and SonarQube to Minikube..."
                    sh 'kubectl --kubeconfig=$KUBECONFIG apply -f student-management/k8s/mysql-deployment.yaml --validate=false'
                    sh 'kubectl --kubeconfig=$KUBECONFIG apply -f student-management/k8s/springboot-deployment.yaml --validate=false'
                    sh 'kubectl --kubeconfig=$KUBECONFIG apply -f student-management/k8s/sonarqube-deployment.yaml --validate=false'

                    sh 'kubectl --kubeconfig=$KUBECONFIG wait --for=condition=ready pod -l app=mysql --timeout=120s'
                    sh 'kubectl --kubeconfig=$KUBECONFIG wait --for=condition=ready pod -l app=springboot --timeout=120s'
                    sh 'kubectl --kubeconfig=$KUBECONFIG wait --for=condition=ready pod -l app=sonarqube --timeout=120s'
                }
            }
        }

        stage('Pipeline Complete') {
            steps {
                echo "CI/CD pipeline finished successfully!"
            }
        }
    }
}
