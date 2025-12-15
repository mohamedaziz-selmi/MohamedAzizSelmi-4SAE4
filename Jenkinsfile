pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }
    stages {

        stage('Clean & Start Minikube') {
    steps {
        echo "Cleaning old Minikube cluster and starting a new one..."
        sh """
            # Delete any existing Minikube cluster
            minikube delete || true

            # Start Minikube with Docker driver
            minikube start --driver=docker --base-image=kicbase/stable:v0.0.48

            # Ensure kubeconfig directory exists for Jenkins user
            mkdir -p /var/lib/jenkins/.kube

            # Set environment variable for kubectl
            export KUBECONFIG=/var/lib/jenkins/.kube/config
        """
    }
}


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

stage('Deploy SonarQube to Minikube') {
    steps {
        echo "Deploying SonarQube on Minikube..."
        dir('student-management/sonarqube-k8s') {
            withEnv(['KUBECONFIG=/var/lib/jenkins/.kube/config']) {
                sh """
                    kubectl apply -f sonarqube-deployment.yaml --validate=false
                    kubectl apply -f sonarqube-service.yaml --validate=false
                    # Wait for SonarQube pod to be ready
                    kubectl wait --for=condition=ready pod -l app=sonarqube --timeout=180s
                """
            }
        }
    }
}

        stage('SonarQube Analysis') {
            steps {
                dir('student-management/student-management') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        echo "Running SonarQube analysis..."
                        sh """
                            SONAR_PORT=\$(kubectl get svc sonarqube -o jsonpath='{.spec.ports[0].nodePort}')
                            SONAR_IP=\$(minikube ip)
                            mvn sonar:sonar \
                                -Dsonar.projectKey=student-management \
                                -Dsonar.projectName=student-management \
                                -Dsonar.host.url=http://\$SONAR_IP:\$SONAR_PORT \
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
                dir('student-management/student-management') {
                    echo "Building Docker image..."
                    sh "docker build -t \$IMAGE_NAME:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management/student-management') {
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

        stage('Deploy Spring Boot App to Kubernetes') {
            steps {
                echo "Deploying Spring Boot app to Minikube..."
                dir('student-management/student-management/k8s') {
                    sh 'kubectl apply -f mysql-deployment.yaml --validate=false'
                    sh 'kubectl wait --for=condition=ready pod -l app=mysql --timeout=180s'

                    sh 'kubectl apply -f springboot-deployment.yaml --validate=false'
                    sh 'kubectl wait --for=condition=ready pod -l app=springboot --timeout=180s'
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
