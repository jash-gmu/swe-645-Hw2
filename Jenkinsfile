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
                    sh 'jar -cvf student.war -C webapp/ .' 
                }
            }
        }
        stage("Build a Docker Image") {
            steps {
                script {
                    // Set DOCKER_BUILDKIT environment variable to enable BuildKit
                    def customImage =docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                        
                    
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
                    sh "kubectl set image deployment/student-deployment student=${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} -n jenkins"

                }
            }
        }
        stage("Deploying Load Balancer in Rancher") {
            steps {
                script {
                    sh "kubectl set image deployment/student-deployment-lb student-deployment-lb=${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} -n jenkins"

                }
            }
        }
    }
}
