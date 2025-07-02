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
    stages {
        stage('increment version') {
            steps {
                script {
                      echo 'increment app version ...'
                      sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} versions:commit'
                      def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                      def version = matcher[0][1]
                      env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    buildImage("kanjamn/demo-app:${IMAGE_NAME}")
                    dockerLogin()
                    dockerPush("kanjamn/demo-app:${IMAGE_NAME}")
                }
            }
        } 
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    echo "THIS IS THE IMAGE NAME: kanjamn/demo-app:${IMAGE_NAME}"
                    def shellCmd = "./server-cmds.sh kanjamn/demo-app:${IMAGE_NAME}"
                    def ec2Instance = "ec2-user@3.70.221.96"

                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} 'chmod +x /home/ec2-user/server-cmds.sh'"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }               
        }
    }
}
