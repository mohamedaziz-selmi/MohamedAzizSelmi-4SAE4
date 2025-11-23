pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'the credentials for docker hub'  // Your Docker Hub global credential ID in Jenkins
        IMAGE_NAME = 'mohamedazizselmi/student-management'
    }

    stages {

        stage('Hello World') {
            steps {
                echo 'Hello world!'
            }
        }

        stage('Maven Version') {
            steps {
                dir('student-management') {  // folder with pom.xml
                    sh 'mvn --version'
                }
            }
        }

        stage('Build Project') {
            steps {
                dir('student-management') {  // folder with pom.xml and Dockerfile
                    echo "Building Maven project..."
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('student-management') {  // Dockerfile is here
                    echo "Building Docker image..."
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management') {
                    echo "Logging in and pushing Docker image..."
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_HUB_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

    }
}
