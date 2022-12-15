pipeline {
    agent any 

    tools {
        maven 'MAVEN3'
    }

    environment{
        NEXUS_URL= "3.138.67.9:8081"
        NEXUS_PROTOCOL = "http"
        NEXUS_VERSION = "1.0-SNAPSHOT"
        

    }

    stages
    {
      stage('Code Download'){

        steps {
            git branch: 'main', credentialsId: 'githublogin', url: 'https://github.com/Faizanvahora120/helloworld-war.git'
        }

      }

      stage('Code Build') {

        steps {
            sh 'mvn clean package -f /var/lib/jenkins/workspace/HelloWolrd-War'
        }
      }

      stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
      }

      stage('Code Quality Check'){
        steps {

            withSonarQubeEnv(installationName: 'sonarserver', credentialsId: 'sonartoken') {
            sh 'mvn sonar:sonar -f /var/lib/jenkins/workspace/HelloWolrd-War/pom.xml'
        }
        }
      }
    
     stage('Artifact Uploading in Nexus Repo')
     {
     steps{
        nexusArtifactUploader artifacts: [[artifactId: 'hello-world-war', classifier: "", file: '/var/lib/jenkins/workspace/HelloWolrd-War/target/hello-world-war-1.0-SNAPSHOT.war', type: 'war']], credentialsId: 'nexuslogin', groupId: 'com.efsavage', nexusUrl: "${NEXUS_URL}" , protocol:"${NEXUS_PROTOCOL}" , nexusVersion: 'nexus3' , repository: 'maven-snapshots', version: "${NEXUS_VERSION}"
     }
     }

    stage('DEV Deploy') {
    steps {
       deploy adapters: [tomcat9(url: 'http://3.19.221.242:8080/', 
                              credentialsId: 'tomcatlogin')], 
                     war: 'target/*.war',
                     contextPath: 'devapp'
        }

    post('Slack Notification'){
        success 
              {
                slackSend channel: 'jenkins-test', failOnError: true, message: 'DEV Deployment was successful; here is the info - Job ["${env.JOB_NAME}${env.BUILD_NUMBER}${env.BUILD_URL}"] ', tokenCredentialId: 'slack'
              }
        failure 
              {
                slackSend channel: 'jenkins-test', failOnError: true, message: 'SORRY - DEV Deployment has failed; here is the info - Job ["${env.JOB_NAME}${env.BUILD_NUMBER}${env.BUILD_URL}"]', tokenCredentialId: 'slack'
              }
                
          }
    }

    stage('Approval Request - QA Deployment'){
      steps{
             timeout(time: 7, unit: 'DAYS') 
              {
               input 'Please approve request for further QA Deployment ?'
              }
           }
      
    }


    stage('QA Deployment') {
    steps {
       deploy adapters: [tomcat9(url: 'http://3.19.221.242:8080/', 
                              credentialsId: 'tomcatlogin')], 
                     war: 'target/*.war',
                     contextPath: 'testapp'
        }

    post('Slack Notification'){
        success 
              {
                slackSend channel: 'jenkins-test', failOnError: true, message: 'QA Deployment was successful. Please continue with testing ; here is the info - Job ["${env.JOB_NAME}${env.BUILD_NUMBER}${env.BUILD_URL}"]', tokenCredentialId: 'slack'
              }
        failure 
              {
                slackSend channel: 'jenkins-test', failOnError: true, message: 'SORRY - QA Deployment was failed; here is the info - Job ["${env.JOB_NAME}${env.BUILD_NUMBER}${env.BUILD_URL}"]', tokenCredentialId: 'slack'
              }
                
            }
        }

    }



  }
