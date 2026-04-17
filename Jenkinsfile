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
                sh 'npm test -- --watchAll=false'
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

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-login',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASS'
                    )]) {
                        sh "docker login -u ${DH_USER} -p ${DH_PASS}"
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DH_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DH_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Trigger deployment pipeline') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        build job: 'Deploy_to_main', wait: false
                        echo "Triggered Deploy_to_main"
                    } else if (env.BRANCH_NAME == 'dev') {
                        build job: 'Deploy_to_dev', wait: false
                        echo "Triggered Deploy_to_dev"
                    } else {
                        error("Unknown branch for deployment: ${env.BRANCH_NAME}")
                    }
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