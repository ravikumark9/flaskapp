pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    agent any
    environment {
        VERSION = 'latest'
        PROJECT = 'flaskapp'
        IMAGE = 'flaskapp:latest'
        ECRURL = 'http://261286523034.dkr.ecr.us-east-1.amazonaws.com/flaskapp'
        ECRCRED = 'ecr:us-east-1:awscred'
        //REGION = "us-east-1"
        //TASK_DEF_URN = "arn:aws:ecs:us-east-1:754901696211:task-definition/first-run-task-definition"
        //CLUSTER = "arn:aws:ecs:us-east-1:754901696211:cluster/default"
        //EXEC_ROLE_URN = "arn:aws:iam::754901696211:role/codedeploy"
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
        stage('Deploy') {
            steps {
                script {
                // Override image field in taskdef file
                //    sh "sed -i 's|{{image}}|${ECRURL}:${commit_id}|' flasktask.json"
                // Create a new task definition revision
               //     sh "aws ecs register-task-definition --execution-role-arn ${EXEC_ROLE_URN} --cli-input-json file://flasktask.json --region ${REGION}"
               
                // Update service on EC2
             //       sh "aws ecs update-service --cluster ${CLUSTER} --service mybni-api-test-service --task-definition ${TASK_DEF_URN} --region ${REGION}"

                     
                        docker.withServer("tcp://172.31.22.94:2375", "dockerserver") {
                            docker.withRegistry("$ECRURL","$ECRCRED") {                            
                              ecrimage = docker.image(IMAGE).pull()
                            
                           //          docker.Image.run('-p 81:80')
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
