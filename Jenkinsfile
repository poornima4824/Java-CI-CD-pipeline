pipeline {
agent any
tools {
    maven 'maven'
}   
    stages {
        stage('Code checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/poornima4824/Java-CI-CD-pipeline.git']]])
                
                } 
            }
        }
        stage('Build') { 
            steps {
                sh "mvn clean package"
            }
        }
    //  stage("Publish to Nexus Repository Manager") {
    //      steps {
    //         script {
    //             def readPom = readMavenPom file: 'pom.xml'
    //             def nexusrepo = readPom.version.endsWith("SNAPSHOT") ? "maven-snapshots" : "maven-releases"
    //             nexusArtifactUploader artifacts: 
    //             [
    //                 [
    //                     artifactId: "${readPom.artifactId}",
    //                     classifier: '', 
    //                     file: "target/${readPom.artifactId}-${readPom.version}.war", 
    //                     type: 'war'
    //                 ]
    //             ], 
    //                     credentialsId: 'nexus', 
    //                     groupId: "${readPom.groupId}", 
    //                     nexusUrl: '52.91.194.232:8081', 
    //                     nexusVersion: 'nexus3', 
    //                     protocol: 'http', 
    //                     repository: "${nexusrepo}", 
    //                     version: "${readPom.version}"

    //         }
    //     }
    // }
    }

}
