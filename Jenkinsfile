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
 stages{
       stage('Increment version') {
         steps {
            script {
              echo 'Incrementing app version..'
               sh 'mvn build-helper:parse-version versions:set \
                 -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                 versions:commit'
            def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
            def version = matcher[0][1]
           env.IMAGE_NAME = "$version-$BUILD_NUMBER"
         }
      }
   }

    stage {
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
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
                    echo "Deploying Docker image to EC2 instance..."
            //        def dockerCmd = 'docker run -p 3080:3080 -d fmstyles/my-app:1.0'
                       def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                       def ec2Instance = "ec2-user@172-31-36-151"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh {ec2Instance}:/home/ec2-user"
                        sh "scp docker-compose.yaml {ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@18.234.124.215 ${shellCmd}"
                    }
                }
            }
        }
         stage("commit version update") {
                steps {
                   script {
                      withCredentials([usernamePassword(credentialsId: 'jenkinsCred', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                          sh 'git config --global user.email "jenkins@example.com" '
                          sh 'git config --global user.name "jenkins" '
                          sh 'git status'
                          sh 'git branch'
                          sh 'git config --list'


                       sh "git remote set-url origin https://${USER}:${PASS}@git@github.com:fmstyles14/Deploy-Application-from-Jenkins-Pipeline-to-EC2-Instance.git"
                       sh 'git add .'
                       sh 'git commit -m "ci:version bump"'
                       sh ' git push origin HEAD:main'
                      }
                   }
                }
          }
    }
}
}
