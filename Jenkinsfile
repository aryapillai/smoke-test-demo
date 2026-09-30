pipeline {

    agent any

    environment {
        AWS_REGION     = "us-east-2"
        AWS_ACCOUNT_ID = "794248399805"
        ECR_REPOSITORY = "smoke-demo"
        IMAGE_URI      = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:latest"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/aryapillai/smoke-test-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t smoke-demo .
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f smoke-test 2>/dev/null || true

                    docker run -d \
                        --name smoke-test \
                        -p 8086:80 \
                        smoke-demo
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    sleep 5
                    curl http://localhost:8086
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login \
                        --username AWS \
                        --password-stdin \
                        $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag smoke-demo:latest $IMAGE_URI
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push $IMAGE_URI
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker rm -f smoke-test 2>/dev/null || true
            '''
        }
    }
}
