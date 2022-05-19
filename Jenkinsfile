//Test1
//Test2
def COMMIT
def BRANCH_NAME
def GIT_BRANCH
pipeline
{
 agent any
 environment
 {
     AWS_ACCOUNT_ID="470022230688"             
     AWS_DEFAULT_REGION="ap-south-1" 
     IMAGE_REPO_NAME="jenkins-java"
     REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
     DOCKERHUB_CREDENTIALS=credentials('docker')
 }
 tools
 {
      maven 'maven3'
 }   

 options 
 {
  buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '4', daysToKeepStr: '', numToKeepStr: '4')
  timestamps()
}
 stages
 {
     stage('Code checkout')
     {
         steps
         {
             script
             {
                 checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vasantha31/jenkins-java-ci-cd-pipeline.git']]])
                 COMMIT = sh (script: "git rev-parse --short=10 HEAD", returnStdout: true).trim()  
                 COMMIT_TAG = sh (script: "git tag --contains | head -1", returnStdout: true).trim() 
            }
             
         }
     }
     stage('Build')
     {   
        //  when {
        //     buildingTag()
        //  }
         steps
         {
             sh "mvn clean package"
         }
     }
    //  stage('Execute Sonarqube Report')
    //  {
    //      steps
    //      {
    //         withSonarQubeEnv('sonar') 
    //          {
    //             sh "mvn sonar:sonar"
    //          }  
    //      }
    //  }
    //  stage('Quality Gate Check')
    //  {
    //      steps
    //      {
    //          timeout(time: 1, unit: 'HOURS') 
    //          {
    //             waitForQualityGate abortPipeline: true
    //         }
    //      }
    //  }
     
    //  stage('Nexus Upload')
    //  {
    //      steps
    //      {
    //         script
    //         {
    //              def readPom = readMavenPom file: 'pom.xml'
    //              def nexusrepo = readPom.version.endsWith("SNAPSHOT") ? "maven-snapshots" : "maven-releases"
    //              nexusArtifactUploader artifacts: 
    //              [
    //                  [
    //                      artifactId: "${readPom.artifactId}",
    //                      classifier: '', 
    //                      file: "target/${readPom.artifactId}-${readPom.version}.war", 
    //                      type: 'war'
    //                  ]
    //             ], 
    //                      credentialsId: 'Nexus-Cred', 
    //                      groupId: "${readPom.groupId}", 
    //                      nexusUrl: '3.110.88.104:8081', 
    //                      nexusVersion: 'nexus3', 
    //                      protocol: 'http', 
    //                      repository: "${nexusrepo}", 
    //                      version: "${readPom.version}"

    //         }
    //      }
    //  }
    
     stage('Docker Build and Tag') {

              steps {

                  sh 'docker build -t sample:latest .'

                  sh 'docker tag  sample vasanthad/sample:latest'

                    }

              }

         stage('Login') {

              steps {

            sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'

                }

              }

          stage('Push') {

                 steps {

                  sh 'docker push  vasanthad/sample:latest'



                 }

           }
     // Stopping Docker containers for cleaner Docker run
     stage('stop previous containers') {
         steps {
            sh 'docker ps -f name=myjavaContainer -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=myjavaContainer -q | xargs -r docker container rm'
         }
       }

    stage('Docker Run') {
     steps{
         script {
                sh "docker run -d -p 8082:8080 --rm --name myjavaContainer sample:latest"
            }
      }
    } 
    //  stage('Update image in K8s manifest file')
    //  {
    //      steps
    //      {
             
    //              sh """#!/bin/bash
    //              sed -i 's/VERSION/$COMMIT/g' deployment.yaml
    //              """
    //          }
    //      }
     
    //  stage('Deploy to K8s cluster')
    //  {
    //      steps
    //      {
             
    //          sh '/usr/local/bin/kubectl apply -f deployment.yaml --record=true'
    //          sh """#!/bin/bash
    //          sed -i 's/$COMMIT/VERSION/g' deployment.yaml
    //          """

    //      }
    //  }
 }

//  post
//  {
//      always
//      {
//          cleanWs()
//      }
//      success
//      {
//         slackSend channel: 'build-notifications',color: 'good', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
//         emailext attachLog: true, body: '''BUILD IS SUCCESSFULL - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
 
//         Check console output at $BUILD_URL to view the results.
 
//         Regards,
 
//         Nithin John George
//         ''', compressLog: true, replyTo: 'njdevops321@gmail.com', 
//         subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: 'njdevops321@gmail.com'
//      }
//      failure
//      {
//          slackSend channel: 'build-notifications',color: 'danger', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
//          emailext attachLog: true, body: '''BUILD IS FAILED - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
 
//         Check console output at $BUILD_URL to view the results.
 
//         Regards,
 
//         Nithin John George
//         ''', compressLog: true, replyTo: 'njdevops321@gmail.com', 
//         subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: 'njdevops321@gmail.com'
//      }
//  }

}
