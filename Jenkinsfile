pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    agent any
    environment {
        VERSION = 'latest'
        PROJECT = 'flaskapp'
        IMAGE = 'flaskapp:latest'
        ECRURL = 'http://634335805137.dkr.ecr.us-east-1.amazonaws.com/flaskapp'
        ECRCRED = 'ecr:us-east-1:awscred'
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
                    docker.build("$IMAGE")
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
    }
    post {
        always {
            // make sure that the Docker image is removed
            sh "docker rmi $IMAGE | true"
        }
    }
} 
