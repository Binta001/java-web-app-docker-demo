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
                nexusArtifactUploader artifacts: [[artifactId: 'java-web-app', classifier: '', file: 'target/java-web-app-5.0.war', type: 'war']], credentialsId: 'nexus-cred', groupId: 'com.mt', nexusUrl: '67.202.27.41:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'nexus-hosted-repo', version: '5.0'
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
	 stage('create ec2') {
             steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "myaws_cred",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                 sh 'aws ec2 run-instances --image-id ami-04581fbf744a7d11f --instance-type t2.medium --security-group-ids sg-0462c95242d0a2d67 --key-name NVkeypair --tag-specifications \'ResourceType=instance,Tags=[{Key=Name,Value=Demoec2}]\' --region us-east-1'
                script {
		def myinstanceid = sh(script: "aws ec2 describe-instances --filters 'Name=tag:Name,Values=Demoec2' --output text --query 'Reservations[*].Instances[*].InstanceId' --region us-east-1", returnStdout: true).trim() 
				println "instance id of the instance is ${myinstanceid}"
				env.INSTANCE_ID = myinstanceid
			}
		
		}
            }
        }
	stage('Get IP') {
             steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "myaws_cred",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                script {
		def myIP = sh(script: "aws ec2 describe-instances --instance-ids ${INSTANCE_ID} --query 'Reservations[].Instances[].PublicIpAddress' --output text --region us-east-1", returnStdout: true).trim() 
				println "IP of the instance is ${myIP}"
				env.IP = myIP
			}
		
		}
            }
        }
	    
	  stage("ssh to ec2") {
                steps {
                    script {
			    sh "chmod 400 NVkeypair.pem && ssh -o StrictHostKeyChecking=no -i NVkeypair.pem ec2-user@${env.IP} 'sudo yum update -y && sudo yum install docker -y && sudo systemctl start docker && sudo usermod -a -G docker ec2-user && sudo chmod 755 /var/run/docker.sock && sudo docker pull fabinta/myjenkins_project:${BUILD_NUMBER} && sudo docker run -td --name mydemocontainer fabinta/myjenkins_project:${BUILD_NUMBER}'"
                    }
                }
        }
		
		
    }
    
        
    
}
