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

                sh '''
                    MINIKUBE_IP=$(minikube ip)
                    SONAR_PORT=$(kubectl get svc sonarqube -o jsonpath="{.spec.ports[0].nodePort}")
                    SONAR_URL="http://$MINIKUBE_IP:$SONAR_PORT"

                    echo "Detected SonarQube URL: $SONAR_URL"

                    mvn sonar:sonar \
                      -Dsonar.projectKey=student-management \
                      -Dsonar.projectName=student-management \
                      -Dsonar.host.url=$SONAR_URL \
                      -Dsonar.login=$SONAR_TOKEN \
                      -DskipTests \
                      -Dsonar.java.binaries=target/classes
                '''
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

   stage('Deploy to Kubernetes') {
    steps {
        echo "Deploying to Kubernetes..."
        // Make sure Jenkins uses the correct kubeconfig
        withEnv(['KUBECONFIG=/var/lib/jenkins/.kube/config']) {
            dir('student-management') {
                // Apply SonarQube manifests
                sh 'kubectl apply -f sonarqube-k8s --validate=false'

                // Apply MySQL secret or manifests if needed
                sh 'kubectl apply -f mysql-secret.yaml --validate=false'

                // Apply other Minikube configs if needed
                sh 'kubectl apply -f minikube-jenkins-config.yaml --validate=false'

                // Wait for pods to be ready
                sh 'kubectl wait --for=condition=ready pod -l app=mysql --timeout=120s || true'
                sh 'kubectl wait --for=condition=ready pod -l app=springboot --timeout=120s || true'
                sh 'kubectl wait --for=condition=ready pod -l app=sonarqube --timeout=120s || true'
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
