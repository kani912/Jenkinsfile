pipeline {
  agent {
    label 'docker'  # separate agent (launched as JAR on host machine) to avoid running docker inside docker
  }
  environment {
    imageId = 'use-name/image-name:1.$BUILD_NUMBER'
    docker_registry = 'your_docker_registry'
    docker_creds = credentials('your_docker_registry_creds')
  }
  stages {
    stage('Docker build') {
      steps {
        sh "docker build --no-cache --force-rm -t ${imageId} ."
      }
    }
    stage('Docker push') {
      steps {
        sh'''
          docker login $docker_registry --username $docker_creds_USR --password $docker_creds_PSW
          docker push $imageId
          docker logout
        '''
      }
    }
    stage('Clean') {
      steps{
        sh "docker rmi ${imageId}"
      }
    }
  }
}
