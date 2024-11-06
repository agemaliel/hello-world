pipeline {
    agent any
    
    environment {
        // Define environment variables
        REPO_URL = 'https://github.com/agemaliel/hello-world.git'
        DOCKER_IMAGE = 'agemaliel/hello-world'
        DOCKER_REGISTRY = 'index.docker.io/v1/'
        DOCKER_CREDENTIAL_ID = 'Dockerhub-Alfon'
        SSH_CREDENTIAL_ID = 'jenkins-ssh'
        REMOTE_USER = 'root'
        REMOTE_HOST = '192.168.147.233'
    }
    
    tools {
        maven 'maven-3.8.5'
    }
    
    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from GitHub using credentials
                git branch: 'main', url: env.REPO_URL
            }
        }
         stage('Build with Maven') {
            steps {
                script {
                    // Run a Maven build (e.g., compile, test, package)
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    def buildNumber = env.BUILD_NUMBER
                    def imageName = "${env.DOCKER_IMAGE}:${buildNumber}"
                    def latestTag = "${env.DOCKER_IMAGE}:latest"
                    def app = docker.build(imageName)
                    //def app = docker.build("${env.DOCKER_IMAGE}:latest")
                    
                    // Tag the image as latest
                    app.tag('latest')
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Log in to Docker registry
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.DOCKER_CREDENTIAL_ID) {
                        // Push Docker image
                        docker.image("${env.DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }
        
        stage('Deploy to Remote Server') {
            steps {
                sshagent(credentials: [env.SSH_CREDENTIAL_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${env.REMOTE_USER}@${env.REMOTE_HOST} 'kubectl create deployment hello-world --image=agemaliel/hello-world:latest && kubectl expose deployment hello-world --type=NodePort --port=8080'
                    """
                }
            }
        }
    }
}