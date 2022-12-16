pipeline {
    agent any 

    tools {
        maven 'MAVEN3'
    }

    environment{
        NEXUS_URL= "3.138.67.9:8081"
        NEXUS_PROTOCOL = "http"
        VERSION = "1.0-SNAPSHOT"
        POM_XML_FILE_PATH = "/var/lib/jenkins/workspace/HelloWolrd-War/pom.xml"
        WAR_FILE_PATH = "/var/lib/jenkins/workspace/HelloWolrd-War/target/hello-world-war-1.0-SNAPSHOT.war"
        ARTIFACT_ID = "hello-world-war"
        ARTIFACT_TYPE = "war"
        CREDENTIALS_ID = "nexuslogin"
        GROUP_ID = "com.efsavage"
        NEXUS_VERSION = "nexus3"
        NEXUS_REPOSITORY = "maven-snapshots"
        TOMCAT_URL = "http://3.19.221.242:8080/"
        TOMCAT_DEV_CONTEXT_PATH = "devapp"
        TOMCAT_TEST_CONTEXT_PATH = "testapp"
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
            sh 'mvn clean package -f "${POM_XML_FILE_PATH}"'
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
            sh 'mvn sonar:sonar -f ${POM_XML_FILE_PATH}"'
        }
        }
      }
    
     stage('Artifact Uploading in Nexus Repo')
     {
     steps{
        nexusArtifactUploader artifacts: [[artifactId: "${ARTIFACT_ID}", classifier: "", file: "${WAR_FILE_PATH}", type: "${ARTIFACT_TYPE}"]], credentialsId: "${CREDENTIALS_ID}", groupId: "${GROUP_ID}", nexusUrl: "${NEXUS_URL}" , protocol:"${NEXUS_PROTOCOL}" , nexusVersion: "${NEXUS_VERSION}" , repository: "${NEXUS_REPOSITORY}", version: "${VERSION}"
     }
     }

    stage('DEV Deploy') {
    steps {
       deploy adapters: [tomcat9(url: "${TOMCAT_URL}", 
                              credentialsId: 'tomcatlogin')], 
                     war: "${WAR_FILE_PATH}",
                     contextPath: "${TOMCAT_DEV_CONTEXT_PATH}"
        }

    post('Slack Notification'){
        success 
              {
                 slackSend(channel:'jenkins-test', message: "DEV Deployment is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                }
        failure 
              {
                 slackSend(channel:'jenkins-test', message: "SORRY - DEV Deployment has failed, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                }
              }
                
          }

    stage('Approval Request - QA Deployment'){
      steps{
             timeout(time: 7, unit: 'DAYS') 
              {
                input message: 'Please approve request for further QA Deployment ?', submitter: 'admin'
              }
           }
      
    }


    stage('QA Deployment') {
    steps {
       deploy adapters: [tomcat9(url: "${TOMCAT_URL}", 
                              credentialsId: 'tomcatlogin')], 
                     war: "${WAR_FILE_PATH}",
                     contextPath: "${TOMCAT_TEST_CONTEXT_PATH}"
        }

    post('Slack Notification'){
        success 
              {
                
                 slackSend(channel:'jenkins-test', message: "QA Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                }
              
        failure 
              {
                 slackSend(channel:'jenkins-test', message: "SORRY - QA Job has failed, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                  
              }
                
            }
        }

    }

  }
