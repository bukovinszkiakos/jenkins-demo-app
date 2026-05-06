pipeline {

    agent any

    environment {
        AWS_REGION = 'eu-central-1'
        ECR_REPO = '554422868760.dkr.ecr.eu-central-1.amazonaws.com/akos-jenkins-demo-app'
        IMAGE_NAME = 'jenkins-demo-app'
        APP_SERVER = '52.29.7.131'
    }

    stages {

        stage('Clone') {
            steps {
                git branch: 'main',
                url: 'https://github.com/bukovinszkiakos/jenkins-demo-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Login to ECR') {
            steps {

                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]]) {

                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {

                sh '''
                docker tag $IMAGE_NAME:latest $ECR_REPO:latest

                docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to App Server') {
            steps {

                sshagent(credentials: ['ec2-ssh-key']) {

                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$APP_SERVER "

                    sudo docker stop app || true
                    sudo docker rm app || true

                    aws ecr get-login-password --region $AWS_REGION | \
                    sudo docker login --username AWS --password-stdin $ECR_REPO

                    sudo docker pull $ECR_REPO:latest

                    sudo docker run -d \
                      --restart unless-stopped \
                      --name app \
                      -p 3000:3000 \
                      $ECR_REPO:latest
                    "
                    '''
                }
            }
        }
    }
}