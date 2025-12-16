pipeline {
    agent any
    tools{
        maven 'mymaven'
    }

    stages {
        stage('code') {
            steps {
                git branch: '$branch', url: 'https://github.com/lankesh-koppisetti/dockerwebapp.git'
            }
        }
        stage('CQA'){
            steps{
                withSonarQubeEnv("mysonar") {
                sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=mysonar -Dsonar.projectName='mysonar'"
                }
            }
        }
        stage ("Build") 
        { 
            steps 
            { 
                sh 'mvn clean package' 
                sh 'cp -r target Docker-app' 
                //sh 'sudo $chmod  777  /var/run/docker.sock' //use if required .......
            } 
        } 
        
        stage('Artifaccts'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.visualpathit', nexusUrl: '54.167.249.98:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'myrepo', version: '4.0.0'
            }
        }
        stage ("Dockerfile")
        { 
            steps 
            { 
                echo 'images created'
            //sh 'docker build -t appimage Docker-app' 
            //sh 'docker build -t dbimage Docker-db' 
            } 
        } 
        
        stage ("Deploy") 
        { 
            input{
                message 'check the branck name and aprove'
            }
            steps 
            { 
                sh 'docker ps -a --filter "name=devopsdb" -q | xargs -r docker stop | xargs -r docker rm '
                sh 'docker ps -a --filter "name=appcont" -q | xargs -r docker stop | xargs -r docker rm '
                sh 'docker run -d --name devopsdb -p 3306:3306 dbimage' 
                sh 'docker run -itd --name appcont -p 1234:8080 --link devopsdb:mysqlcon appimage' 
            } 
        } 
       
    }
     post {
            always {
                mail to: 'lankeshtest33@gmail.com,k.lankesh33@gmail.com',
                subject: "pipeline status: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
                }
         }
}
