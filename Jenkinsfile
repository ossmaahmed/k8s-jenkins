pipeline {
    agent any

    environment {
        APP_NAME = 'k8s-jenkins'
        REPO_URL = "https://github.com/ossmaahmed/k8s-jenkins.git"
    }

    stages {
        stage('Getting Repo files') {
            steps {
                git branch: 'main', credentialsId: 'jenkins-toknen', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t "$APP_NAME:$BUILD_NUMBER" .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        set +x
                        DOCKER_CONFIG=$(mktemp -d)
                        export DOCKER_CONFIG
                        trap 'rm -rf "$DOCKER_CONFIG"' EXIT
                        printf '%s' "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker tag "$APP_NAME:$BUILD_NUMBER" "$DOCKER_USERNAME/$APP_NAME:$BUILD_NUMBER"
                        docker push "$DOCKER_USERNAME/$APP_NAME:$BUILD_NUMBER"
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        set +x
                        kubectl create secret docker-registry docker-registry \
                            --docker-server=https://index.docker.io/v1/ \
                            --docker-username="$DOCKER_USERNAME" \
                            --docker-password="$DOCKER_PASSWORD" \
                            --dry-run=client -o yaml | kubectl apply -f -

                        sed "s|my-image|$DOCKER_USERNAME/$APP_NAME:$BUILD_NUMBER|" \
                            k8s/deployment.yaml | kubectl apply -f -
                        kubectl apply -f k8s/svc.yaml
                        kubectl rollout status deployment/my-deployment --timeout=120s
                    '''
                }
            }
        }
    }
}