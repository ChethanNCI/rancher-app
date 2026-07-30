pipeline {
    agent {
        kubernetes {
            defaultContainer 'kaniko'
        }
    }

    environment {
        REGISTRY = "44.202.77.70:30002"
        IMAGE = "rancher/myapp"
        DEPLOYMENT = "rancher-apps"
        NAMESPACE = "default"
    }

    stages {

        stage('Configure Harbor Auth') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'harbor-jenkins',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASS'
                    )
                ]) {
                    sh '''
                        mkdir -p /kaniko/.docker

                        cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "${REGISTRY}": {
      "username": "${HARBOR_USER}",
      "password": "${HARBOR_PASS}"
    }
  }
}
EOF
                    '''
                }
            }
        }

        stage('Build and Push') {
            steps {
                sh '''
                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=$WORKSPACE \
                      --destination=$REGISTRY/$IMAGE:$BUILD_NUMBER \
                      --insecure
                '''
            }
        }

        stage('Test Deploy Script') {
            steps {
                container('kubectl') {
                    sh '''
                        cd "$WORKSPACE"

                        echo "Current directory:"
                        pwd

                        echo "Workspace contents:"
                        ls -la

                        echo "Replacing BUILD_NUMBER in app.yaml..."
                        sed -i "s/BUILD_NUMBER/${BUILD_NUMBER}/g" app.yaml

                        echo "Updated app.yaml:"
                        cat app.yaml

                        echo "Shell script executed successfully."
                        echo "Skipping Kubernetes deployment."
                    '''
                }
            }
        }
    }
}