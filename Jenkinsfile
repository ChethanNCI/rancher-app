pipeline {

    agent {
        kubernetes {
            defaultContainer 'jnlp'
        }
    }


    environment {
        REGISTRY   = "44.202.77.70:30002"
        IMAGE      = "rancher/myapp"
        DEPLOYMENT = "rancher-apps"
        NAMESPACE  = "default"
    }


    stages {


        stage('Checkout') {

            steps {
                echo "Source code checkout completed"
            }
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


                        AUTH=$(printf "%s:%s" "$HARBOR_USER" "$HARBOR_PASS" | base64 | tr -d '\\n')


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


        stage('Verify Harbor Login') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'harbor-jenkins',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASS'
                    )
                ]) {

                    sh '''
                        echo "Testing Harbor authentication..."

                        curl \
                        -u "$HARBOR_USER:$HARBOR_PASS" \
                        -I \
                        http://${REGISTRY}/v2/

                    '''
                }
            }
        }



        stage('Verify Kaniko Configuration') {

            steps {

                sh '''

                    echo "Checking Kaniko..."

                    ls -la /kaniko

                    echo "Checking Docker auth file..."

                    ls -la /kaniko/.docker


                    echo "Docker config:"
                    cat /kaniko/.docker/config.json | sed 's/"auth": "[^"]*"/"auth": "HIDDEN"/g'

                '''
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
                      --insecure \
                      --skip-tls-verify \
                      --verbosity=debug


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

                    kubectl apply \
                    -f app.yaml \
                    -n ${NAMESPACE}



                    echo "Waiting for rollout..."

                    kubectl rollout status \
                    deployment/${DEPLOYMENT} \
                    -n ${NAMESPACE}


                    echo "Deployment completed successfully."

                '''
            }
        }

    }


    post {

        success {
            echo "Pipeline completed successfully"
        }


        failure {
            echo "Pipeline failed"
        }

    }

}