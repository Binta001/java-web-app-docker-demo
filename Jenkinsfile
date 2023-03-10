pipeline{
    agent any
    environment {
        PATH = "$PATH:/usr/share/maven/bin"
        DOCKERHUB_CREDENTIALS = credentials('dockerhubcred')
    }
    stages{
        stage("cloning repo"){
            steps{
              git 'https://github.com/Binta001/java-web-app-docker-demo.git'
            }
        }
        stage("building code"){
             steps{
                sh 'mvn clean install'
             }
        }
       stage('SonarQube analysis') {
            steps{
                  withSonarQubeEnv('sonarserver') { 
                  sh "mvn sonar:sonar"
					}
			}
       }
       stage('Upload artifact to Nexus repo') {
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'java-web-app', classifier: '', file: 'target/java-web-app-5.0.war', type: 'war']], credentialsId: 'nexus-cred', groupId: 'com.mt', nexusUrl: '34.227.173.186:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'nexus-hosted-repo', version: '5.0'
                }
               
        }
       stage("Build Docker Image") {
            steps{
               sh "docker build -t fabinta/javarepo:${BUILD_NUMBER} ."
            }
        }
        stage("Docker Push"){
            steps{
            sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            sh "docker push fabinta/javarepo:${BUILD_NUMBER}"
            }
        }
        
    }
}
