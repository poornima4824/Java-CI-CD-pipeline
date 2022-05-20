pipeline
{
 agent any
 environment {
     jmeter="/opt/jmeter/bin"
     DOCKERHUB_CREDENTIALS=credentials('doc-pri')
     AWS_CREDENTIALS= credentials('AWS-cred')
     AWS_ACCOUNT_ID="435255528170"             
     AWS_DEFAULT_REGION="us-east-1" 
     IMAGE_REPO_NAME="java-app"
     REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
   }
 tools
 {
      maven 'maven'
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
                 checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/poornima4824/Java-CI-CD-pipeline.git']]])
                 COMMIT = sh (script: "git rev-parse --short=10 HEAD", returnStdout: true).trim()  
                 COMMIT_TAG = sh (script: "git tag --contains | head -1", returnStdout: true).trim() 
            }
             
         }
     }
     stage('Build')
     { 
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
      stage('Jmeter test') {
         steps {
               sh "/opt/jmeter/bin/jmeter.sh -Jjmeter.save.saveservice.output_format=xml -n -t src/main/jmeter/Testing.jmx -l MyRun1.jtl"
               step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
              //sh "/opt/jmeter/bin/jmeter -Jjmeter.save.saveservice.output_format=xml -n -t src/main/jmeter/Testing.jmx -l src/main/jmeter/JMeter.jtl -e -o src/main/jmeter/report/output"
              //sh "/opt/jmeter/bin/jmeter.sh -Jjmeter.save.saveservice.output_format=html -Jjmeter.save.saveservice.output_format=xml -n -t src/main/jmeter/Testing.jmx -l src/main/jmeter/MyRun1.jtl"
             step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl,**/*.html'])
             //sh "mvn clean verify"
                   
         }
     }
    stage('Publish Report') {
            steps {
            
                perfReport filterRegex: '', sourceDataFiles: '**/*.jtl,**/*.html'
            
            }
        }
     
    //  stage("Publish to Nexus Repository Manager") {
    //         steps {
    //              script
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
    //                      credentialsId: 'nexus', 
    //                      groupId: "${readPom.groupId}", 
    //                      nexusUrl: '52.91.194.232:8081', 
    //                      nexusVersion: 'nexus3', 
    //                      protocol: 'http', 
    //                      repository: "${nexusrepo}", 
    //                      version: "${readPom.version}"

    //         }
    //      }
    //  }
    
    //  stage('Docker Build and Tag') {

    //           steps {

    //             //   sh 'docker build -t java-app:latest .'
    //             //   sh 'docker tag  java-app nagapoornima/java-app:latest'
    //             sh 'docker build -t java-web-app:latest .'
    //             sh 'docker tag  java-web-app boppanaaadhya/java-web-app:latest'

    //                 }

    //           }

        //  stage('Login') {
        //       steps {
        //     sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        //         }
        //       }
        //   stage('Push') {
        //          steps {
        //         //   sh 'docker push  nagapoornima/java-app:latest'
        //          sh 'docker push boppanaaadhya/java-web-app'
        //          }
        //    }

         stage('Login to AWS ECR')
      {
          steps
          {
              script
              {
                 sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 435255528170.dkr.ecr.us-east-1.amazonaws.com"
                 //sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                 // sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 440883647063.dkr.ecr.us-east-1.amazonaws.com"
              }
          }
      }
      stage('Building Docker Image')
       {
           steps
           {
               script
               {
                 sh "docker build -t java-app ."
                //sh "docker build . -t ${REPOSITORY_URI}:LoginWebApp"
               }
           }
       }
       stage('Pushing Docker image into ECR')
       {
           steps
           {
             script
              {
                sh "docker tag java-app:latest 435255528170.dkr.ecr.us-east-1.amazonaws.com/java-app:latest"
                sh "docker push 435255528170.dkr.ecr.us-east-1.amazonaws.com/java-app:latest"
                //  sh "docker push ${REPOSITORY_URI}:sample-login-app"
              }
           }

     }
     // Stopping Docker containers for cleaner Docker run
     stage('stop previous containers') {
         steps {
            sh 'docker ps -f name=myjavaapp -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=myjavaapp -q | xargs -r docker container rm'
         }
       }

    stage('Deploy') {
     steps{
         script {
                sh "docker run -d -p 8084:8080 --rm --name myjavaapp java-app:latest"
            }
      }
    } 
    
 }
 post
 {
     success
     {
        slackSend channel: 'build-notifications',color: 'good', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
        emailext attachLog: true, body: '''BUILD IS SUCCESSFULL - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS: Check console output at $BUILD_URL to view the results. /n
 
         Regards,
         Team
 
     ''', compressLog: true, replyTo: 'naga.poornima22@gmail.com', 
        subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: 'naga.poornima22@gmail.com'
     }
     failure
     {
         slackSend channel: 'build-notifications',color: 'danger', message: "started  JOB : ${env.JOB_NAME}  with BUILD NUMBER : ${env.BUILD_NUMBER}  BUILD_STATUS: - ${currentBuild.currentResult} To view the dashboard (<${env.BUILD_URL}|Open>)"
         emailext attachLog: true, body: '''BUILD IS FAILED - $PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS:
 
         Check console output at $BUILD_URL to view the results. 
        Regards,
        Team
         ''', compressLog: true, replyTo: 'naga.poornima22@gmail.com', 
         subject: '$PROJECT_NAME - $BUILD_NUMBER - $BUILD_STATUS', to: 'naga.poornima22@gmail.com'
      }
  }

}
