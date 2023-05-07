/**
@Library('lokeshsharedlibs') _
pipeline {
    agent any
    
    stages{
    
    stage('checkout code') {
        steps{
            sendSlackNotifications('STARTED')
            git credentialsId: 'github-credentials', url: 'https://github.com/lokeshsgithub/maven-web-application.git'
        }
    }

    }//stages closed
    post{
        success{
            sendSlackNotifications(currentBuild.result)
        }
        failure{
            sendSlackNotifications(currentBuild.result)
        }
    }
}//pipeline closed
**/
pipeline {
    agent any

    tools{
        maven 'maven3.9.0'
    }
    
    environment {
        buildNumber = "BUILD_NUMBER"
    }

    stages{

        stage('checkout code'){
            steps{
                git credentialsId: 'github-credentials', url: 'https://github.com/lokeshsgithub/maven-web-application.git'
            }
        }
        
        stage('Unit test: Maven'){
            steps{
                sh "mvn test"
            }
        }

        stage('Integration of testcases: Maven'){
            steps{
                sh "mvn verify -DskipunitTests"
            }
        }

        stage('Building the package: Maven'){
            steps{
                sh "mvn clean install"
            }
        }
        
        stage('code quality: sonarqube'){
            steps{
                withSonarQubeEnv(credentialsId: 'sonarqube_credentials',installationName: 'sonarqube') {
                 sh "mvn clean package sonar:sonar"                       
              }
            }
        }
        
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        
        stage('Build the docker image'){
            steps{
                sh "docker build -t lokeshsdockerhub/mavenwebapp:$BUILD_NUMBER ."
            }
        }

        stage('scan the image: Trivy'){
            steps{
                sh "trivy image lokeshsdockerhub/mavenwebapp:$BUILD_NUMBER"
            }
        }

        stage('Push the image into Registry'){
            steps{
                sh "docker push lokeshsdockerhub/mavenwebapp:$BUILD_NUMBER"
            }
        }
        
        
    }//staged closed
}//pipeline closed