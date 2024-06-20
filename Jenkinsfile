pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                // Example build step
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Example test step
                sh 'make test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Example deploy step
                sh 'make deploy'
            }
        }
    }
}
