pipeline {

    agent {
        kubernetes {
            defaultContainer 'jnlp'
        }
    }

    environment {
        REGISTRY = "44.202.77.70:30002"
        IMAGE = "rancher/myapp"
        DEPLOYMENT = "rancher-apps"
        NAMESPACE = "default"
    }

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
                echo "Configuring Harbor authentication..."

                mkdir -p /kaniko/.docker

                AUTH=$(echo -n "${HARBOR_USER}:${HARBOR_PASS}" | base64)

                cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "${REGISTRY}": {
      "auth": "${AUTH}"
    }
  }
}
EOF

                echo "Harbor authentication configured."
            '''
        }
    }
}


        stage('Build and Push Image') {

            steps {

                sh '''
                    echo "Building and pushing image..."

                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=$WORKSPACE \
                      --destination=$REGISTRY/$IMAGE:$BUILD_NUMBER \
                      --insecure

                    echo "Image pushed successfully."
                '''
            }
        }


        stage('Deploy Application') {

            steps {

                sh '''
                    echo "Updating Kubernetes manifest..."

                    sed -i "s/BUILD_NUMBER/${BUILD_NUMBER}/g" app.yaml


                    echo "Generated Kubernetes manifest:"
                    cat app.yaml


                    echo "Deploying application..."

                    kubectl apply -f app.yaml \
                      -n ${NAMESPACE}


                    echo "Waiting for rollout..."

                    kubectl rollout status deployment/${DEPLOYMENT} \
                      -n ${NAMESPACE}


                    echo "Deployment completed successfully."
                '''
            }
        }
    }
}