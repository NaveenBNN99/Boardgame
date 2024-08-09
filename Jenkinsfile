pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID') 
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        IMAGE_REPO_NAME = "boardgame"
        IMAGE_TAG = "latest"
        REPOSITORY_URI = "654654434841.dkr.ecr.us-east-1.amazonaws.com/boardgame:latest"
    }
    
    stages {
        stage('Authenticate') {
            steps {
                script {
                    
                    def ecrAuthToken = sh(script: "aws ecr get-login-password --region ${AWS_DEFAULT_REGION}", returnStdout: true).trim()
                    
                    
                    sh "docker login -u AWS -p ${ecrAuthToken} 654654434841.dkr.ecr.us-east-1.amazonaws.com/boardgame"
                    
                   
                }
            }
        }
        
        stage('Cloning Git') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/NaveenBNN99/Boardgame.git']])
            }
        }
        
        stage('Build with mvn') {
            steps {
                script {
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }
        
        stage('Pushing to ECR') {
            steps {
                script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
                }
            }
        }

        stage('Install kubectl') {
            steps {
                script {
                    def kubectlDir = "/usr/local/bin/"  
                    env.PATH = "${kubectlDir}:${env.PATH}"
                }
            }
        }

        stage("Deploy to EKS") {
            steps {
                script {
                    echo "Current PATH: ${env.PATH}"
                    sh "aws eks update-kubeconfig --name boardgame"
                    sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
    }
}
