pipeline {
    agent any

    stages {

        stage('Checkout Test') {
            steps {
                echo 'Starting Jenkins build test...'
                echo 'Repository checkout successful'
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "Checking Jenkins environment"
                    echo "Current user:"
                    whoami
                    echo "Current directory:"
                    pwd
                    echo "Files:"
                    ls -la
                '''
            }
        }

        stage('Build Test') {
            steps {
                echo 'Running sample build process...'

                sh '''
                    echo "Compiling application..."
                    sleep 5
                    echo "Build completed successfully"
                '''
            }
        }

        stage('Unit Test') {
            steps {
                echo 'Running unit tests...'

                sh '''
                    echo "Executing test cases..."
                    sleep 3
                    echo "All tests passed"
                '''
            }
        }

        stage('Deployment Simulation') {
            steps {
                echo 'Simulating deployment...'

                sh '''
                    echo "Deploying application..."
                    sleep 3
                    echo "Deployment completed"
                '''
            }
        }
    }

    post {

        success {
            echo '✅ Jenkins pipeline completed successfully!'
        }

        failure {
            echo '❌ Jenkins pipeline failed!'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}