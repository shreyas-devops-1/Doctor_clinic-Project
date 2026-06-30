pipeline {
    agent any

    environment {
        IMAGE_NAME   = "yourdockerhubuser/clinic-frontend"
        IMAGE_TAG    = "${env.BUILD_NUMBER}"
        DEPLOY_HOST  = "deploy@your-server-ip"
        DEPLOY_PATH  = "/opt/clinic-app"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Lint / Sanity check') {
            steps {
                // Placeholder until real test suite exists.
                // Once backend is built, run: npm ci && npm test here.
                sh 'echo "No automated tests yet — add npm test stage when backend exists"'
            }
        }

        stage('Build Docker image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy to server') {
            steps {
                sshagent(credentials: ['clinic-deploy-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_HOST} '
                            cd ${DEPLOY_PATH} &&
                            docker compose pull &&
                            docker compose up -d --remove-orphans &&
                            docker image prune -f
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployed build ${IMAGE_TAG} successfully."
        }
        failure {
            echo "Build ${IMAGE_TAG} failed — check logs above."
        }
    }
}
