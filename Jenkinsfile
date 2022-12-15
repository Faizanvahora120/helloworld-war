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
            sh 'mvn clean package -f /var/lib/jenkins/workspace/helloworld-war/pom.xml'
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
            sh 'mvn sonar:sonar -f /var/lib/jenkins/workspace/helloworld-war/pom.xml'
        }
        }
      }
    
     stage('Artifact Uploading in Nexus Repo')
     {
     steps{
        nexusArtifactUploader artifacts: [[artifactId: 'hello-world-war', classifier: "", file: '/var/lib/jenkins/workspace/helloworld-war/target/*.war', type: 'war']], credentialsId: 'nexuslogin', groupId: 'com.efsavage', nexusUrl: "${NEXUS_URL}" , protocol:"${NEXUS_PROTOCOL}" , nexusVersion: 'nexus3' , repository: 'maven-snapshots', version: "${NEXUS_VERSION}"
     }
     }

    stage('DEV Deploy') {
    steps {
       deploy adapters: [tomcat9(url: 'http://3.19.221.242:8666/', 
                              credentialsId: 'tomcatlogin')], 
                     war: 'target/*.war',
                     contextPath: 'demoapp'
        }
    }

    }
}