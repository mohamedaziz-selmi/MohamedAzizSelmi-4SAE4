pipeline {
    agent any
    environment {
        IMAGE_NAME      = 'mohamedazizselmi/student-management'
        DOCKER_TAG      = "${env.BUILD_NUMBER}"
        DEPLOYMENT_NAME = 'studentmang-app'
        NAMESPACE       = 'default'
        KUBECONFIG      = '/var/lib/jenkins/.kube/config'
        SONAR_PORT      = '9000'
    }
    stages {
        stage('Setup SonarQube Port Forward') {
            steps {
                script {
                    sh "pkill -f 'port-forward.*sonarqube' || true"
                    sh """
                        nohup kubectl port-forward svc/sonarqube -n ${NAMESPACE} 9000:9000 > /tmp/sonar-pf.log 2>&1 &
                        echo \$! > /tmp/sonar-pf.pid
                        sleep 5
                    """
                    env.SONAR_URL = "http://localhost:9000"
                    echo "SonarQube URL: ${env.SONAR_URL}"

                    sh """
                    for i in {1..10}; do
                        if curl -f --max-time 5 ${env.SONAR_URL}/api/system/status 2>/dev/null; then
                            echo "SonarQube reachable!"
                            exit 0
                        fi
                        echo "Attempt \$i failed, retrying..."
                        sleep 2
                    done
                    echo "WARNING: Cannot reach SonarQube"
                    """
                }
            }
        }

        stage('Verify Kubernetes') {
            steps {
                sh '''
                echo "Minikube status:"; minikube status
                echo "Current context:"; kubectl config current-context
                echo "Cluster info:"; kubectl cluster-info
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    rm -rf student-management
                    git clone -b main git@github.com:fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git student-management
                '''
            }
        }

        stage('Maven Build') {
            steps {
                dir('student-management/student-management') {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('student-management/student-management') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.projectName=student-management \
                            -Dsonar.host.url=${env.SONAR_URL} \
                            -Dsonar.login=$SONAR_TOKEN \
                            -DskipTests \
                            -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                dir('student-management/student-management') {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        docker build -t ${IMAGE_NAME}:${DOCKER_TAG} -t ${IMAGE_NAME}:latest .
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}:${DOCKER_TAG}
                        docker push ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('student-management/student-management/k8s') {
                    withEnv(["KUBECONFIG=$KUBECONFIG"]) {
                        sh '''
                        for f in *.yaml; do
                            echo "Applying $f..."
                            kubectl apply -f "$f" --validate=false
                        done

                        for label in mysql springboot sonarqube; do
                            kubectl wait --for=condition=ready pod -l app=$label --timeout=180s || true
                        done
                        kubectl get pods -n ${NAMESPACE}
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
            sh "echo SonarQube URL: ${env.SONAR_URL}/dashboard?id=student-management"
        }
        failure {
            echo "❌ Pipeline failed!"
            sh '''
            echo "=== Minikube Status ==="; minikube status || true
            echo "=== Kubectl Context ==="; kubectl config current-context || true
            echo "=== Cluster Info ==="; kubectl cluster-info || true
            '''
        }
    }
}
