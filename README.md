#!/usr/bin/env groovy
def imageId = "use-name/image-name:1.$BUILD_NUMBER"
pipeline {
    agent {
        label 'docker'  # separate agent (launched as JAR on host machine) to avoid running docker inside docker
    }
    stages {
        stage('Test') {
            steps {
                script {
                    sh "docker build --no-cache --target test -t ${imageId} ."
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    sh "docker build --target build -t ${imageId} ."
                }
            }
        }
        stage('Image') {
            steps {
                script {
                    sh "docker build --target final -t ${imageId} ."
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry('' , 'dockerhub') {
                        dockerImage = docker.build("${imageId}")
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Clean') {
          steps{
            sh "docker rmi ${imageId}"
          }
        }
    }
}

   
