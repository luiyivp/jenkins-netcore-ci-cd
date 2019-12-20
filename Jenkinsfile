pipeline {
    agent none
    stages {
        stage('Build Project') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('SonarQube Analysis') {
            agent any
            environment {
                sonarqubeScannerHome = tool 'sonar'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://sonarqube:9000 -Dsonar.projectName=jenkins-ci-cd -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=cicd -Dsonar.sources=src/main/java/com/mycompany/app"
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Publish Artifact') {
            agent any
            steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "http://artifactory:8081/artifactory",
		            credentialsId: 'artifactory'
                )
                rtUpload (
                    serverId: 'ARTIFACTORY_SERVER',
                    spec: '''{
                        "files": [
                            {
                            "pattern": "target/my-app-1.0-SNAPSHOT.jar",
                            "target": "jenkins/"
                            }
                        ]
                    }'''
                )
            }
        }
        stage('Build Image') {
            steps {
                script {
                    javaAppImage = docker.build("java-cicd:${env.BUILD_ID}")
                }
            }
        }
        /*stage('Push Image') {
            agent any
            steps {
                rtDockerPush(
                    serverId: "ARTIFACTORY_SERVER",
                    image: "artifactory:8081/" + "java-cicd:${env.BUILD_ID}",
                    //host: 'tcp://localhost:2375',
                    targetRepo: 'docker-repo', // where to copy to (from docker-virtual)
                    properties: 'project-name=java-cicd;status=stable'
                )
            }
        }*/
    }
}
