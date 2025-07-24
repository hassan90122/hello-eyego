pipeline {
  agent any

  environment {
    ECR_REPO = '471112811957.dkr.ecr.us-east-1.amazonaws.com/hello-eyego'
    AWS_REGION = 'us-east-1'
    IMAGE_TAG = 'latest'
  }

  stages {
    stage('Github Pull') {
      steps {
        git 'https://github.com/hassan90122/hello-eyego.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          dockerImage = docker.build("${ECR_REPO}:${IMAGE_TAG}")
        }
      }
    }

    stage('Authenticate to AWS ECR') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh '''
            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
            aws configure set default.region $AWS_REGION
            aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPO
          '''
        }
      }
    }

    stage('Push Image to ECR') {
      steps {
        script {
          dockerImage.push("${IMAGE_TAG}")
        }
      }
    }

      stage('Deploy to EKS') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh '''
            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
            aws configure set default.region $AWS_REGION

            aws eks update-kubeconfig --region $AWS_REGION --name hello-eyego-cluster

            kubectl apply -f deployment.yaml
            kubectl apply -f service2.yaml
            kubectl apply -f ingress.yaml
          '''
        }
      }
    }
  }
}
