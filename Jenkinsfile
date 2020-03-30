pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }
    agent any
    environment {
        VERSION = 'latest'
        PROJECT = 'ecr-ecs'
        IMAGE = 'ecr-ecs:latest'
        ECRURL = 'http://906996567172.dkr.ecr.us-east-1.amazonaws.com/ecr-ecs'
        ECRCRED = 'ecr:us-east-1:awscred'
        REGION = "us-east-1"
        TASK_DEF_URN = "arn:aws:ecs:us-east-1:906996567172:task-definition/first-run-task-definition"
        CLUSTER = "arn:aws:ecs:us-east-1:906996567172:cluster/ecr-ecs"
       // EXEC_ROLE_URN = "arn:aws:iam::380552115531:role/ecsTaskExecutionRole"
        FAMILY = "first-run-task-definition"
        NAME = "ecr-ecs"
        SERVICE_NAME = "ecr-ecs-service"
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
               //  Override image field in taskdef file
                    sh "sed -i 's|{{image}}|${ECRURL}:${commit_id}|' flasktask.json"
               //  Create a new task definition revision
               //     sh "aws ecs register-task-definition --execution-role-arn ${EXEC_ROLE_URN} --cli-input-json file://flasktask.json --region ${REGION}"
                      sh "aws ecs register-task-definition --family ${FAMILY} --cli-input-json file://flasktask.json --region ${REGION}"
                   //   SERVICES=`aws ecs describe-services --services ${SERVICE_NAME} --cluster ${CLUSTER} --region ${REGION} | jq.failures[]`
                   //	  REVISION=`aws ecs describe-task-definition --task-definition ${NAME} --region ${REGION} | jq.taskDefiniton.revision`
               // Get latest version
	              //   REVISION = sh "aws ecs describe-task-definition --task-definition ${NAME} --region ${REGION} | "egrep "revision" | tr "/" " " | awk '{print $2}' | sed 's/"$//'"
                      REVISION = sh "aws ecs describe-task-definition --task-definition ${NAME} --region ${REGION} |  script: "egrep "revision" | tr "/" " " | awk '{print $2}' | sed 's/"$//'", returnStdout: true).trim()""

		       //    Update service on EC2
                   // sh "aws ecs update-service --cluster ${CLUSTER} --service ecr-ecs-service --task-definition ${TASK_DEF_URN} --region ${REGION}"
                      sh "aws ecs update-service --cluster ${CLUSTER} --region ${REGION} --service ${SERVICE_NAME} -- task-definiton ${FAMILY}:${REVISION}"
                 //     sh "aws ecs create-service --service-name ${SERVICE_NAME} --launch-type FARGATE --desired-count 1 --task-definition ${FAMILY} --cluster ${CLUSTER} --region ${REGION}"
                     
                       // docker.withServer("tcp://172.31.22.94:2375", "dockerserver") {
                       //     docker.withRegistry("$ECRURL","$ECRCRED") {                            
                      //         docker.image(IMAGE).pull()
                            
                           //          docker.Image.run('-p 81:80')
                     //          ecrimage = sh(script: 'docker images | awk {\'print $3\'} | head -2 | tail -1', returnStdout: true).trim()
                     //          sh "docker run -d -p 8081:8081 ${ecrimage}"
                      //      }
                  // }
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
