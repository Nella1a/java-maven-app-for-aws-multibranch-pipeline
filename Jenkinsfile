#!/usr/bin/env groovy

library identifier: 'jenkins-shared-lib-jmaven-app-for-aws@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://github.com/Nella1a/jenkins-shared-lib-jmaven-app-for-aws.git',
    credentialsID: 'github-credentials'
    ]
)

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    environment {
        IMAGE_NAME = 'kanjamn/demo-app:java-maven-1.0'
    }
    stages {
        stage('build app') {
            steps {
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def shellCmd = "bash ./server-cmds.sh"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ec2-user@3.70.221.96:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@3.70.221.96:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.70.221.96 ${shellCmd}"
                    }
                }
            }               
        }
    }
}
