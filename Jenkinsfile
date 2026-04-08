pipeline {
    agent any

    environment {
        IMAGE_NAME = "deniskolyachko/task15-app"
        DEV_CONTAINER = "task15-dev"
        PROD_CONTAINER = "task15-prod"
        DEV_PORT = "8081"
        PROD_PORT = "8082"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    env.FULL_IMAGE = "${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $FULL_IMAGE .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $FULL_IMAGE'
            }
        }

        stage('Deploy to DEV') {
            when {
                branch 'dev'
            }
            steps {
                sh '''
                    docker rm -f $DEV_CONTAINER || true
                    docker run -d --name $DEV_CONTAINER -p $DEV_PORT:80 $FULL_IMAGE
                '''
            }
        }

        stage('Deploy to PROD') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    docker rm -f $PROD_CONTAINER || true
                    docker run -d --name $PROD_CONTAINER -p $PROD_PORT:80 $FULL_IMAGE
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                    docker image prune -af || true
                    docker container prune -f || true
                    docker builder prune -af || true
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
