pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    agent any
    environment {
        VERSION = 'latest'
        PROJECT = 'ecr-ecs'
        IMAGE = 'ecr-ecs:latest'
        ECRURL = 'http://429489393157.dkr.ecr.us-east-1.amazonaws.com/ecr-ecs'
	ECRURN = '429489393157.dkr.ecr.us-east-1.amazonaws.com/ecr-ecs'
        ECRCRED = 'ecr:us-east-1:awscred'
        REGION = "us-east-1"
        TASK_DEF_URN = "arn:aws:ecs:us-east-1:429489393157:task-definition/first-run-task-definition"
        CLUSTER = "arn:aws:ecs:us-east-1:429489393157:cluster/ecr-ecs"
	EXEC_ROLE_URN = "arn:aws:iam::429489393157:role/ecsTaskExecutionRole"
        FAMILY = "first-run-task-definition"
        NAME = "ecr-ecs"
        SERVICE_NAME = "ecr-ecs-service"
    }
    stages {
      stage('Build preparations') {
            steps {
                script {
                    // calculate GIT lastest commit short-hash
                    
                  //  commit_id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                 //hi   
                 //   VERSION = "${PROJECT}_${commit_id}"
                  
                    
                    //VERSION = "${BUILD_ID}${VERSION}"
		    VERSION = "${PROJECT}_${BUILD_ID}"
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
               //  Override image field in taskdef file
                    sh "sed -i 's|{{image}}|${ECRURN}:ecr-ecs_${BUILD_ID}|' flasktask.json"
               //  Create a new task definition revision
               //     sh "aws ecs register-task-definition --execution-role-arn ${EXEC_ROLE_URN} --cli-input-json file://flasktask.json --region ${REGION}"
                    sh "aws ecs register-task-definition  --execution-role-arn ${EXEC_ROLE_URN} --family ${FAMILY} --cli-input-json file://flasktask.json --region ${REGION}"
               // Get latest version
	       //       REVISION=$(aws ecs describe-task-definition --task-definition ${FAMILY} --region ${REGION} | jq '.taskDefinition.revision')
		    REVISION = sh (
                            script: "aws ecs describe-task-definition --task-definition ${FAMILY} --region ${REGION} | jq '.taskDefinition.revision'",
                            returnStdout: true
                    ).trim()
		
		       sh "aws ecs update-service --force-new-deployment --cluster ${CLUSTER} --region ${REGION} --service ${SERVICE_NAME} --task-definition ${FAMILY}:${REVISION} --desired-count 1"
                  //    sh "aws ecs create-service --service-name ${SERVICE_NAME} --launch-type FARGATE --desired-count 1 --task-definition ${FAMILY} --cluster ${CLUSTER} --region ${REGION}"
                     
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
