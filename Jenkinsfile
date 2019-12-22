pipeline {
    agent none
    stages {
        stage('Release project processes') {
            agent {
                docker {
                    image 'mcr.microsoft.com/dotnet/core/sdk:3.1'
                    args '-u root:root'
                }
            }
            stages {
                stage('Restore packages Project') {
                    steps {
                        sh 'dotnet restore src/netcore-api/netcore-api.csproj'
                    }
                }
                stage('Clean project') {
                    steps {
                        sh 'dotnet clean src/netcore-api/netcore-api.csproj'
                    }
                }
                stage('Build project') {
                    steps {
                        sh 'dotnet build src/netcore-api/netcore-api.csproj'
                    }
                }
                stage('Package project') {
                    steps {
                        sh 'dotnet pack src/netcore-api/netcore-api.csproj'
                    }
                }
            }
        }
        stage('SonarQube analysis') {
            agent any
            environment {
                sonarqubeScannerHome = tool 'sonar'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://sonarqube:9000 -Dsonar.scm.exclusions.disabled=true -Dsonar.projectName=netcore-api -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=netcore -Dsonar.sources=src/netcore-api"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Publish artifact') {
            agent any
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'nexus:8081',
                    groupId: 'api',
                    version: '1.0',
                    repository: 'dotnet-releases',
                    credentialsId: 'nexus',
                    artifacts: [
                        [artifactId: 'netcoreAPI',
                        classifier: '',
                        file: 'src/netcore-api/bin/Debug/netcore-api.1.0.0.nupkg',
                        type: 'nupkg']
                    ]
                )
            }
        }
    }
}