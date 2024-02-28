pipeline {
    agent any

    environment {
        DOCKERFILE_PATH = 'docker/Dockerfile'
        DOCKER_IMAGE_NAME = 'jashwanthraj/webapp'
        registry='jashwanthraj/webapp'
        DOCKER_REGISTRY_CREDENTIALS = 'docker' 
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
    }

    stages {
        stage("Build a War File") {
            steps {
                script {
                    checkout scm
                    sh 'rm -rf *.war'
                    sh 'jar -cvf docker/student.war -C webapp/ .' 
                }
            }
        }
        stage("Build a Docker Image") {
            steps {
                script {
                    // Set DOCKER_BUILDKIT environment variable to enable BuildKit
                   
                    dockerImage = docker.build registry:${env.BUILD_NUMBER}
                    // Use Docker Buildx to build the Docker image
                    
                }
            }
        }
        stage("Push the Docker Image to Docker Hub") {
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_REGISTRY_CREDENTIALS}") {
                        docker.image("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
        stage ("Deploying Image as a Single Node to Rancher") {
            steps {
                script {
                    sh 'kubectl set image deployment/student-deployment student=${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} -n jenkins pipeline'
                }
            }
        }
        stage("Deploying Load Balancer in Rancher") {
            steps {
                script {
                    sh 'kubectl set image deployment/student-deployment-lb student=${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} -n jenkins pipeline'
                }
            }
        }
    }
}
