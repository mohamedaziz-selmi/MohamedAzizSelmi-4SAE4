pipeline {
    agent any
    stages {
        stage('Hello World') {  // Exemple 1 (Page 7)
            steps {
                echo 'Hello world !'
            }
        }
        stage('Git Clone') {  // Exemple 2 (Page 7) - avec credentialsId pour privé
            steps {
                git url: 'https://github.com/fourth-git-copilot-account/MohamedAzizSelmi-4SAE4.git',
                    branch: 'main',
                    credentialsId: 'd53472d1-7c06-4517-893b-219f23f95bc3'  // Remplace par ton ID (ex: github-credentials)
            }
        }
        stage('Maven Version') {  // Exemple 3 (Page 7)
            steps {
                sh 'mvn --version'
            }
        }
    }
}
