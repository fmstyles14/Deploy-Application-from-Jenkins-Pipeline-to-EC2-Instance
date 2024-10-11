#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever:modernSCM(
    [$class:'GitSCMSource',
    remote: 'https://github.com/fmstyles14/jenkins-shared-library.git',
    credentialsId:'jenkinsCred'])

pipeline {
    agent any
    tools {
        maven 'maven-3.9.9'
    }
    environment {
        IMAGE_NAME = 'fmstyles/my-app:1.0'
    }
    stages {
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 

        stage("deploy") {
            steps {
                script {
                    echo "Deploying the application..."
                    def dockerCmd = 'docker run -p 3080:3080 -d fmstyles/my-app:1.0'
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@18.234.124.215 ${dockerCmd}"
                    }
                }
            }
        }               
    }
} 
