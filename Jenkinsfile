pipeline {
    agent none
    stages {
        stage('Build Project') {
            agent {
                docker {
                    image 'mcr.microsoft.com/dotnet/core/sdk:3.1'
                    args '-u root:root'
                }
            }
            steps {
                sh 'pwd'
                sh 'ls -ltr'
            }
        }
    }
}