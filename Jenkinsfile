pipeline {
    agent any
    environment {
        DOCKERHUB_CRED = credentials('dockerhub-cred')
        IMAGE_NAME = "${env.DOCKERHUB_CRED_USR}/cicd-pipeline"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        EC2_HOST = "3.96.128.130" // Replace with deployment target
        SSH_KEY = credentials('ec2-ssh-key')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/jacksongeorge770/Terraform-Automation.git', branch: 'main', credentialsId: 'GIT-LOGIN'
            }
        }
        stage('Build GoLang') {
            steps {
                sh 'go mod download'
                sh 'go build -o webserver ./main.go'
            }
        }
        stage('Build Docker') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        stage('Push Docker') {
            steps {
                sh 'echo $DOCKERHUB_CRED_PSW | docker login -u $DOCKERHUB_CRED_USR --password-stdin'
                sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ubuntu@$EC2_HOST "
                        docker pull ${IMAGE_NAME}:${IMAGE_TAG}
                        docker stop webserver || true
                        docker rm webserver || true
                        docker run -d --name webserver -p 8000:8000 ${IMAGE_NAME}:${IMAGE_TAG}
                    "
                '''
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}