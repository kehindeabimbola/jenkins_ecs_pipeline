pipeline {
  environment {
    registry = "elrrdockerhub/kenny-api"
    registryCredential = 'dockerhub_creds'
    dockerImage = ''
  }
  agent any

  stages {
    stage('Git Checkout') {
      steps {
        git credentialsId: 'github_kenny_creds', url: 'https://github.com/kehindeabimbola/jenkins_ecs_pipeline.git'
      }
    }
    stage('Test') {
      steps {
        sh 'npm install'
        sh 'npm test'
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Push Image to DockerHub') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
            dockerImage.push('latest')
          }
        }
      }
    }
    stage('Cleaning up') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
    stage('Deploy Image to ECS') {
      steps{
        withAWS(credentials: 'aws_creds') {
          sh "ecs deploy --image softramscloud_api_container docker.io/elrrdockerhub/kenny-api:latest SoftramsCloud-ECS-Cluster softramscloud-api --region us-east-1 --access-key-id $AWS_ACCESS_KEY_ID --secret-access-key $AWS_SECRET_ACCESS_KEY  --rollback --timeout 900"
        }
      }
    }    
  }
}
