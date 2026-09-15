pipeline {

    agent any

    parameters {
        choice(
            name: 'IMAGE_TAG',
            choices: ['1.0', '1.1', '1.2'],
            description: 'Docker image version'
        )
    }

    environment {
        AWS_REGION = 'ap-southeast-2'
        AWS_ACCOUNT_ID = '600766011991'
        ECR_REPOSITORY = 'team-java-app'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_URI = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
    }

    stages {

        stage('Checkout') {

            steps {
                checkout scm
            }
        }

        stage('Maven Build') {

            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {

            steps {
                sh 'docker build -t team-java-app:${IMAGE_TAG} .'
            }
        }

        stage('ECR Login') {

            steps {

                sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Docker Tag') {

            steps {

                sh '''
                    docker tag \
                    team-java-app:${IMAGE_TAG} \
                    ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('Docker Push') {

            steps {

                sh '''
                    docker push \
                    ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {

        success {

            echo "Image ${ECR_URI}:${IMAGE_TAG} pushed successfully."
        }

        failure {

            echo "Pipeline failed."
        }
    }
}