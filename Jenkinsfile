pipeline {
    agent any

    environment {
        IMAGE_NAME = "cicd-pipeline:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        CONTAINER_NAME = "cicd-pipeline-${env.BRANCH_NAME}"
    }

    stages {
        stage('Set environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.APP_PORT = '3001'
                    } else {
                        error("Unsupported branch: ${env.BRANCH_NAME}")
                    }
                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Port: ${env.APP_PORT}"
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh '''
                        docker rm -f ${CONTAINER_NAME} || true
                        docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:3000 ${IMAGE_NAME}
                    '''
                }
            }
        }
    }
}