pipeline {
    agent any

    environment {
        REGISTRY = "44.202.77.70:30002"
        IMAGE = "rancher/myapp"
    }

    stages {

        stage('Build') {
            steps {
                container('docker') {
                    sh '''
                    docker --version
                    docker build -t $REGISTRY/$IMAGE:$BUILD_NUMBER .
                    '''
                }
            }
        }

        stage('Login') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'harbor-creds',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASS'
                    )]) {
                        sh '''
                        echo "$HARBOR_PASS" | docker login $REGISTRY \
                        --username "$HARBOR_USER" \
                        --password-stdin
                        '''
                    }
                }
            }
        }

        stage('Push') {
            steps {
                container('docker') {
                    sh '''
                    docker push $REGISTRY/$IMAGE:$BUILD_NUMBER
                    '''
                }
            }
        }
    }
}