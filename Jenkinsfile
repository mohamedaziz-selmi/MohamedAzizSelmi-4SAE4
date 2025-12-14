pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mohamedazizselmi/student-management'
        SONAR_URL = 'http://192.168.49.2:31666'
    }
    stages {
        stage('Clone repo via SSH') {
            steps {
                echo "Cloning repository via SSH..."
                sh 'git clone git@github.com:fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git student-management'
            }
        }

        stage('Maven Build') {
            steps {
                dir('student-management') {
                    echo "Building project..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('student-management') {
                    echo "Running SonarQube analysis..."
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.login=\$SONAR_TOKEN \
                            -DskipTests
                        """
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                dir('student-management') {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                        docker build -t \$IMAGE_NAME:latest .
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push \$IMAGE_NAME:latest
                        """
                    }
                }
            }
        }
    }
}
