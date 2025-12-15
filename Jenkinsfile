pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_URL = 'http://192.168.49.2:31666'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }
    stages {

        stage('Clean & Start Minikube') {
            steps {
                echo "Cleaning old Minikube cluster and starting a new one..."
                sh '''
                    minikube delete || true
                    minikube start --driver=docker --base-image=kicbase/stable:v0.0.48
                    mkdir -p $(dirname $KUBECONFIG)
                    export KUBECONFIG=$KUBECONFIG
                '''
            }
        }

        stage('Checkout code via SSH') {
            steps {
                echo "Cloning repository via SSH..."
                sh '''
                    rm -rf student-management
                    git clone -b main git@github.com:fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git student-management
                '''
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
                            -Dsonar.host.url=$SONAR_URL \
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
                    sh "docker build -t $IMAGE_NAME:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management/student-management') {
                    echo "Logging in and pushing Docker image..."
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push $IMAGE_NAME:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('student-management/student-management/k8s') {
                    echo "Applying Kubernetes manifests..."
                    // Debug: see which files are in the folder
                    sh 'ls -l'
                    withEnv(["KUBECONFIG=$KUBECONFIG"]) {
                        sh '''
                            kubectl apply -f mysql-deployment.yaml --validate=false
                            kubectl apply -f springboot-deployment.yaml --validate=false
                            kubectl apply -f sonarqube-deployment.yaml --validate=false

                            kubectl wait --for=condition=ready pod -l app=mysql --timeout=180s
                            kubectl wait --for=condition=ready pod -l app=springboot --timeout=180s
                            kubectl wait --for=condition=ready pod -l app=sonarqube --timeout=180s
                        '''
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
