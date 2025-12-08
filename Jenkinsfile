pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-creds'
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_TOKEN = credentials('sonar-token')
        KUBE_WAIT_TIMEOUT = '180s'
        MAX_RETRIES = 3
        RETRY_DELAY = 10
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

        stage('Detect Services') {
            steps {
                script {
                    // Automatically detect Minikube IP
                    MINIKUBE_IP = sh(script: "minikube ip", returnStdout: true).trim()
                    SONAR_URL = "http://${MINIKUBE_IP}:31001"
                    echo "Detected SonarQube URL: ${SONAR_URL}"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    echo "Checking SonarQube connectivity..."
                    retry(env.MAX_RETRIES) {
                        sh "curl --fail -I ${SONAR_URL} || (echo 'SonarQube unreachable, retrying...' && sleep ${RETRY_DELAY} && false)"
                    }

                    echo "Running SonarQube analysis..."
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.projectName=student-management \
                            -Dsonar.host.url=${SONAR_URL} \
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
                    sh 'docker build -t ${IMAGE_NAME}:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management') {
                    echo "Checking Docker Hub connectivity..."
                    retry(env.MAX_RETRIES) {
                        sh "curl --fail https://registry-1.docker.io/v2/ || (echo 'Docker Hub unreachable, retrying...' && sleep ${RETRY_DELAY} && false)"
                    }

                    echo "Logging in and pushing Docker image..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG_CREDENTIAL', variable: 'KUBECONFIG')]) {
                    echo "Deploying to Kubernetes..."
                    sh 'kubectl config use-context minikube'
                    sh 'kubectl get nodes'

                    echo "Applying deployments..."
                    sh 'kubectl apply -f student-management/k8s/mysql-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/springboot-deployment.yaml --validate=false'
                    sh 'kubectl apply -f student-management/k8s/sonarqube-deployment.yaml --validate=false'

                    echo "Waiting for pods to be ready..."
                    sh "kubectl wait --for=condition=ready pod -l app=mysql --timeout=${KUBE_WAIT_TIMEOUT}"
                    sh "kubectl wait --for=condition=ready pod -l app=springboot --timeout=${KUBE_WAIT_TIMEOUT}"
                    sh "kubectl wait --for=condition=ready pod -l app=sonarqube --timeout=${KUBE_WAIT_TIMEOUT}"
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
