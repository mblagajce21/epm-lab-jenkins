pipeline {
    agent any

    tools {
        nodejs 'Node-7.8.0'
    }

    environment {
        BRANCH = "${env.BRANCH_NAME}"
        IMAGE_NAME = "node${env.BRANCH_NAME}"
        IMAGE_TAG  = "v1.0"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Check out branch: ${BRANCH}"
            }
        }

        stage('Set Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.APP_PORT    = '3000'
                        env.LOGO_SOURCE = 'logos/logo-main.svg'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.APP_PORT    = '3001'
                        env.LOGO_SOURCE = 'logos/logo-dev.svg'
                    } else {
                        error("Unknown branch: ${env.BRANCH_NAME}")
                    }
                    echo "Branch=${BRANCH}, Port=${env.APP_PORT}"
                }
            }
        }

        stage('Change Logo') {
            steps {
                sh "cp ${env.LOGO_SOURCE} src/logo.svg"
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                echo "Build complete"
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --watchAll=false --passWithNoTests'
                echo "Tests passed"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def fullTag = "${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker build --build-arg APP_PORT=${env.APP_PORT} -t ${fullTag} ."
                    echo "Docker image built: ${fullTag}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def containerName = "app-${BRANCH}"
                    def fullTag       = "${IMAGE_NAME}:${IMAGE_TAG}"

                    sh """
                        if [ \$(docker ps -aq -f name=^${containerName}\$) ]; then
                            docker stop ${containerName} || true
                            docker rm   ${containerName} || true
                            echo "Removed old container: ${containerName}"
                        fi
                    """

                    sh """
                        docker run -d \\
                            --name ${containerName} \\
                            -p ${env.APP_PORT}:${env.APP_PORT} \\
                            -e PORT=${env.APP_PORT} \\
                            ${fullTag}
                    """
                    echo "Deployed ${fullTag}: http://localhost:${env.APP_PORT}"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded for branch: ${BRANCH}"
        }
        failure {
            echo "Pipeline failed for branch: ${BRANCH}"
        }
    }
}