pipeline {
    agent any

    parameters {
        string(name: 'AWS_CREDENTIAL_ID', defaultValue: 'markwang access', description: 'The ID of the AWS credentials to use')
        string(name: 'ECS_CLUSTER', defaultValue: 'ecscluster11120', description: 'The name of the ECS cluster')
        string(name: 'ECS_SERVICE', defaultValue: 'backend-ecs-service', description: 'The name of the ECS service')
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'The Git branch to build and deploy')
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-southeast-2'
        DOCKER_IMAGE_NAME = 'techscrum'
        DOCKER_IMAGE_TAG = 'latest'
        DOCKERFILE_PATH = '.'
        ECR_REPO_URL = '842279851978.dkr.ecr.ap-southeast-2.amazonaws.com/backend_ecr' // Update with your ECR repository URL

    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the specified branch from GitHub
                    checkout([$class: 'GitSCM', branches: [[name: params.GIT_BRANCH]], userRemoteConfigs: [[url: 'https://github.com/shangchewang/DevOps-Techscrum.be.git']]])
                }
            }
        }
        stage('build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                sh 'npm run lint'
            }
        }

        stage('Test') {
            steps {
                sh 'npm run test'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} ${env.DOCKERFILE_PATH}" // Build Docker image
                }
            }
        }

        stage('Docker Push to ECR') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: params.AWS_CREDENTIAL_ID,
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh "aws ecr get-login-password --region ${env.AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${env.ECR_REPO_URL}" // Authenticate with ECR
                        sh "docker tag ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} ${env.ECR_REPO_URL}:${env.DOCKER_IMAGE_TAG}" // Tag for AWS ECR
                        sh "docker push ${env.ECR_REPO_URL}:${env.DOCKER_IMAGE_TAG}" // Push to AWS ECR
                    }
                }
            }
        }

        stage('Deploy to ECS cluster') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                        credentialsId: params.AWS_CREDENTIAL_ID]
                    ]) {
                    script {
                        sh "aws ecs update-service --cluster ${params.ECS_CLUSTER} --service ${params.ECS_SERVICE} --force-new-deployment"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
            slackSend (color: '#00FF00', message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            echo 'Deployment failed!'
            slackSend (color: '#FF0000', message: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
}
