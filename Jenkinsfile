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

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                    cd "$WORKSPACE"

                    echo "Current directory:"
                    pwd

                    ls -la

                   kubectl apply -f "$WORKSPACE/account.yaml"
                   
                   sed -i "s/BUILD_NUMBER/${BUILD_NUMBER}/g" "$WORKSPACE/app.yaml"
                   
                   kubectl apply -f "$WORKSPACE/app.yaml" -n ${NAMESPACE}
                   
                   kubectl rollout status deployment/${DEPLOYMENT} -n ${NAMESPACE}
                    '''
                }
            }
        }
    }
}