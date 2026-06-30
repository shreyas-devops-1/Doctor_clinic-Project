pipeline {
    agent any

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

        stage('Sanity check') {
            steps {
                // Placeholder until a real test suite exists.
                sh 'echo "No automated tests yet"'
            }
        }

        stage('Build and Deploy') {
            steps {
                // Same machine, so no docker push / no SSH needed.
                // docker compose builds the image AND restarts the containers in one go.
                sh 'docker compose down'
                sh 'docker compose up -d --build'
            }
        }

        stage('Health check') {
            steps {
                // wait a moment for nginx to come up, then verify it responds
                sh 'sleep 5'
                sh 'curl -f http://localhost || (echo "Site did not respond" && exit 1)'
            }
        }
    }

    post {
        success {
            echo "Deployed build ${env.BUILD_NUMBER} successfully on this server."
        }
        failure {
            echo "Build ${env.BUILD_NUMBER} failed — check logs above."
            sh 'docker compose logs --tail=50'
        }
    }
}
