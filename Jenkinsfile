pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://192.168.49.2:31666'
        SONAR_PROJECT_KEY = 'student-management'
        SONAR_PROJECT_NAME = 'student-management'
        DOCKER_IMAGE = 'mohamedazizselmi/student-management:latest'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'  // path to Jenkins user's kubeconfig
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Checking out repository...'
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                dir('student-management/student-management') {
                    echo 'Building Maven project...'
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    dir('student-management/student-management') {
                        echo 'Running SonarQube analysis...'
                        sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN} \
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
                    echo 'Building Docker image...'
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('student-management/student-management') {
                    echo 'Pushing Docker image...'
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('student-management/student-management') {
                    echo 'Deploying to Kubernetes...'
                    sh """
                    export KUBECONFIG=${KUBECONFIG}
                    kubectl --context=minikube apply -f k8s/
                    """
                }
            }
        }

        stage('Done') {
            steps {
                echo 'Pipeline finished!'
            }
        }

    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
