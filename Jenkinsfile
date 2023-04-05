pipeline {
    agent any
    
    environment {
        PATH = "$PATH:/usr/share/maven/bin"
        DOCKERHUB_CREDENTIALS = credentials('DOCKERHUB_CREDENTIALS')
    }

    stages {
        stage('cloning repo') {
            steps {
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
                  withSonarQubeEnv('sonar_server') { 
                  sh "mvn sonar:sonar"
					}
			}
       }
       
       stage('Upload artifact to Nexus') {
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'java-web-app', classifier: '', file: 'target/java-web-app-5.0.war', type: 'war']], credentialsId: 'nexus-cred', groupId: 'com.mt', nexusUrl: '3.95.90.55:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'nexus-hosted-repo', version: '5.0'
                }
               
        }
        
        stage("Build Docker Image")  {
            steps{
               sh "docker build -t fabinta/myjenkins_project:${BUILD_NUMBER} ."
            }
        }
        stage("Docker push"){
            steps{
            sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            sh "docker push fabinta/myjenkins_project:${BUILD_NUMBER}"
            }
        }
	       
	   stage('downloadkey') {
              steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "myaws_cred",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                 sh 'aws s3 cp s3://cf-templates-1gv4da4p4r1b7-us-east-1/NVkeypair.pem .'
                }
            }
        }
	 stage('create stack') {
             steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "myaws_cred",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                 sh 'aws ec2 run-instances --image-id ami-00c39f71452c08778 --instance-type t2.medium --key-name NVkeypair --tag-specifications \'ResourceType=instance,Tags=[{Key=Name,Value=Demoec2}]\' --region us-east-1'
                }
            }
        }
		
		
    }
    
        
    
}
