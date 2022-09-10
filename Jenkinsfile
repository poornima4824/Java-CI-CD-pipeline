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
                    user = "poc"
                    host = "3.144.132.81"
                    sshagent(credentials : ['remote']) {
                        sh 'ssh -o StrictHostKeyChecking=no $user@$host uptime'
                        sh 'ssh -v $user@$host'
                        withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                           sh "mkdir rollback && wget --user=$USERNAME --password=$PASSWORD 'http://3.23.126.255:8081/repository/Rollback_mechanism/build/build_artifact/${GIT_COMMIT}/build_artifact-${GIT_COMMIT}.war'"
                           sh 'cd rollback && java -jar build_artifact-${GIT_COMMIT}.war'
                        }
                    }

                }
            }
        }
    }

}
