pipeline {
    
    agent any
    
    tools {
         maven 'Maven3'
    }
   
    stages {
        
        stage('Checkout') {
            steps {
            checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'f0e6fc7b-1d0b-4c91-be84-fb98b3cb36ab', url: 'https://github.com/olayusuf22/Jobweekslab']]])
            }
        }
        
        stage ('Build') {
            steps {
              sh 'mvn clean install -f MyWebApp/pom.xml'
            }
        }
    
        stage ('Code Quality') {
             steps {
                     withSonarQubeEnv('Sonarqube') {
                     sh  'mvn  sonar:sonar -f MyWebApp/pom.xml'
                }
            }   
        }

        stage ('JaCoCo') {
         steps {
         jacoco()
        }
    }
    
        stage ('Nexus Upload') {
      steps {
      nexusArtifactUploader(
      nexusVersion: 'nexus3',
      protocol: 'http',
      nexusUrl: 'http://ec2-23-22-38-29.compute-1.amazonaws.com:8081',
      groupId: 'myGroupId',
      version: '1.0-SNAPSHOT',
      repository: 'maven-snapshots',
      credentialsId: '7b1cf78f-6d29-42c9-94b8-a3bc250d126f',
      artifacts: [
      [artifactId: 'MyWebApp',
      classifier: '',
      file: 'MyWebApp/target/MyWebApp.war',
      type: 'war']
      ])
      }
    }
    
          stage ('DEV Deploy') {
         steps {
         echo "deploying to DEV Env "
         deploy adapters: [tomcat9(credentialsId: 'a5b93065-f1b1-4b36-ba7d-203453098948', path: '', url: 'http://ec2-54-165-96-243.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
        }
    }
    
         stage ('Slack Notification') {
         steps {
          echo "deployed to DEV Env successfully"
          slackSend(channel:'jobweekslab', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
    
             stage ('DEV Approve') {
              steps {
              echo "Taking approval from DEV Manager for QA Deployment"
              timeout(time: 7, unit: 'DAYS') {
              input message: 'Do you want to deploy?', submitter: 'admin'
            }
        }
    }
     
         stage ('QA Deploy') {
          steps {
          echo "deploying to QA Env "
          deploy adapters: [tomcat9(credentialsId: 'a5b93065-f1b1-4b36-ba7d-203453098948', path: '', url: 'http://ec2-54-165-96-243.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
        }
    }
    
              stage ('QA Approve') {
             steps {
             echo "Taking approval from QA manager"
              timeout(time: 7, unit: 'DAYS') {
              input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
            }
        }
    }
    
               stage ('Slack Notification for QA Deploy') {
               steps {
               echo "deployed to QA Env successfully"
               slackSend(channel:'jobweekslab', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }  
    }
}
