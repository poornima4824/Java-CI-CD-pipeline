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
        stage('Upload Artifacts To Nexus') {
            steps {
                script {
                    nexusArtifactUploader artifacts:
                    [
                        [
                            artifactId: 'build_artifact',
                            classifier: '',
                            file: "target/maven-web-application-0.0.1-SNAPSHOT.war",
                            type: 'war'
                        ]
                    ],
                    credentialsId: 'nexus',
                    groupId: 'build',
                    nexusUrl: '3.144.132.81:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'Rollback_mechanism',
                    version: "${GIT_COMMIT}"
                }
            }
        }
       stage ('Deploy') {
           steps {
               script {
                    remote = [:]
                    remote.name = 'deploy'
                    remote.host = "3.144.132.81"
                    remote.allowAnyHosts = true
                    remote.failOnError = withCredentials([usernamePassword(credentialsId: 'remotehost', password: 'Private Key', usernameVariable: 'username')]) {
                        remote.user = username
                        remote.password = Private Key
                        //sshCommand remote: remote, command: 'mkdir test'
                        withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            sshCommand remote: remote, command: "mkdir rollback && wget --user=$USERNAME --password=$PASSWORD 'http://3.144.132.81:8081/repository/Rollback_mechanism/build/build_artifact/${GIT_COMMIT}/build_artifact-${GIT_COMMIT}.war'"
                            sshCommand remote: remote, command: 'cd rollback && java -jar build_artifact-${GIT_COMMIT}.war'
                        }
                    }
               }
           }
       }
    }

}
