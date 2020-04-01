pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    agent any
    environment {
        VERSION = 'latest'
        PROJECT = 'ecr-ecs'
        IMAGE = 'ecr-ecs:latest'
        ECRURL = 'http://916167090451.dkr.ecr.us-east-1.amazonaws.com/ecr-ecs'
        ECRCRED = 'ecr:us-east-1:awscred'
        REGION = "us-east-1" 
    }
    stages {
      stage('Build preparations') {
            steps {
                script {
                    // calculate GIT lastest commit short-hash
                    
                    commit_id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    
                    VERSION = "${PROJECT}_${commit_id}"
                  
                    
                    //VERSION = "${BUILD_ID}${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    // Build the docker image using a Dockerfile
                    docker.build("$IMAGE","--no-cache .")
                }
            }
        }
        stage('Docker push') {
            steps {
                script {
                    // login to ECR 
                    docker.withRegistry("$ECRURL","$ECRCRED") {
                    docker.image(IMAGE).push()
                   }
                }
            }
         }
        stage('Deploy') {
            steps {
               script {
             
                        docker.withServer("tcp://172.31.22.94:2375", "dockerserver") {
                            docker.withRegistry("$ECRURL","$ECRCRED") {                            
                               docker.image(IMAGE).pull()                          
                           //    docker.Image.run('-p 81:80')
                               sh '''#!/bin/bash
                               a=$(docker ps | grep 8081 | awk '{print $1}')
                               if [ "$a" == "" ];then
                               echo "no container running"
                               else
                               docker stop $a
                               fi
                               '''
                               ecrimage = sh(script: 'docker images | awk {\'print $3\'} | head -2 | tail -1', returnStdout: true).trim()
                               sh "docker run -d -p 8081:8081 ${ecrimage}"
                            }
                   }
                }
            }
       } 
    }
    post {
        always {
            // make sure that the Docker image is removed
            sh "docker rmi $IMAGE | true"
        }
    }
} 
