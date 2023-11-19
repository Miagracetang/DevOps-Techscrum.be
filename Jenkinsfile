pipeline {
    agent any

    parameters {
        string(name: 'AWS_CREDENTIAL_ID', defaultValue: 'miagracetang-at-145367278427', description: 'The ID of the AWS credentials to use')
        string(name: 'ECS_CLUSTER', defaultValue: 'mia-ecs-cluster', description: 'The name of the ECS cluster')
        string(name: 'ECS_SERVICE', defaultValue: 'mia-ecs-service', description: 'The name of the ECS service')
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'The Git branch to build and deploy')
    }

    environment {
        AWS_DEFAULT_REGION = 'ap-southeast-2'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Miagracetang/DevOps-Techscrum.be.git'
            }
        }

    stages {
        stage('test AWS credentials') {
            steps {
                withAWS(credentials: '8e63c1fe-a242-4779-bddc-17569e9888c8', region: 'ap-southeast-2') {
                    s3DoesObjectExist bucket: 'be-jenkins-miagracetang', path: 'hello.txt'
                }
            }
        }
    }

        stage('build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                sh 'npm run lint'
                sh 'docker builder prune -f'
            }
        }

        stage('Test') {
            steps {
                sh 'npm run test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t techscrum:lastest ."
                }
            }
        }

 
        stage('Deploy to ECS') {
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