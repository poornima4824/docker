pipeline {
agent any
 environment
 {
     AWS_ACCOUNT_ID="805392809179"             
     AWS_DEFAULT_REGION="us-east-2" 
     IMAGE_REPO_NAME="poc2"
     REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
     
 }
    stages {
        stage('Code checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-accesstoken', url: 'https://github.com/demo-organization-01/ozontel.git']]])
                }
            }
        } 
        stage('Building Docker Image') {
            steps {
                script {
                  sh "docker build -f Dockerfile -t ${REPOSITORY_URI}:${GIT_COMMIT} ."
               }
            }
        }
        
        stage('Login to AWS ECR') {
            steps {
                script {
                   // sh "aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 805392809179.dkr.ecr.us-east-2.amazonaws.com"
                    sh "aws ecr get-login-password --region us-east-2 | docker login -u AWS --password-stdin 805392809179.dkr.ecr.us-east-2.amazonaws.com"
                }
            }
        }
        stage('Pushing Docker image into dockerhub') {
            steps {
                script {
                    sh "docker build -t poc2 ."
                    sh "docker tag poc2:latest ${REPOSITORY_URI}:${GIT_COMMIT}"
                    sh "docker push ${REPOSITORY_URI}:${GIT_COMMIT}"
                }
            }

        }
        stage('stop previous containers') {
            steps {
                sh 'docker ps -f name=sample -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -fname=sample -q | xargs -r docker container rm'
         }
       }

        stage('Deploy') {
            steps {
                script {
                    try {
                        sh "docker run -d -p 8082:8080 --rm --name sample ${REPOSITORY_URI}:${GIT_COMMIT}"
      
                        sh "docker container inspect -f '{{.State.Status}}' sample  > status "
                        sh '''if grep 'exited' status; then exit 1; else true; fi '''
                    }
                    catch (e) {
                        echo 'Rolling Back to Previous Sucessfull Version'
                        sh "docker run -d -p 8083:8080 --rm --name sample ${REPOSITORY_URI}:${GIT_PREVIOUS_SUCCESSFUL_COMMIT}"
                    }      
                 
                   }

                }
            }
      
    }
} 
