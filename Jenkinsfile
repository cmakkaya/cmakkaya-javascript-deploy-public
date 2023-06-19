pipeline {
    agent any
    environment {
        ECR_REGISTRY="592681249053.dkr.ecr.us-east-1.amazonaws.com"
        APP_REPO_NAME="cmakkaya/cmakkaya-javascript-deploy"
        AWS_REGION="us-east-1"
        PATH="/usr/local/bin/:${env.PATH}"
    }
    stages {
        stage('Create ECR Repo') {            
            steps {
                echo "Creating ECR Repo for nodejs app"
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin "$ECR_REGISTRY"'
                sh '''
                aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
                         aws ecr create-repository \
                         --repository-name ${APP_REPO_NAME} \
                         --image-scanning-configuration scanOnPush=true \
                         --image-tag-mutability IMMUTABLE \
                         --region ${AWS_REGION}
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build --force-rm -t "$ECR_REGISTRY/$APP_REPO_NAME:latest" .'
                sh 'docker image ls'
            }
        }
        stage('Push Image to ECR Repo') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin "$ECR_REGISTRY"'
                sh 'docker push "$ECR_REGISTRY/$APP_REPO_NAME:latest"'
            }
        }
        stage('Deploy') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin "$ECR_REGISTRY"'
                sh 'docker pull "$ECR_REGISTRY/$APP_REPO_NAME:latest"'
                sh 'docker rm -f todo | echo "there is no docker container named todo"'
                sh 'docker run --name todo -dp 80:3000 "$ECR_REGISTRY/$APP_REPO_NAME:latest"'
            }
        }

    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}