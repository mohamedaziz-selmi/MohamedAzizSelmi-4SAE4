pipeline {
    agent any
    stages {
        stage('Hello World') {
            steps {
                echo 'Hello world !'
            }
        }
        stage('Maven Version') {
            steps {
                sh 'mvn --version'
            }
        }
        stage('Build Project') {
            steps {
                dir('student-management') {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }
    }
}
