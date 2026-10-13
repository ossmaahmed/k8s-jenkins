pipeline {
    agent any

    environment {
        APP_NAME = 'k8s-jenkins'
        REPO_URL = "https://github.com/ossmaahmed/k8s-jenkins.git"
    }

    stages {
        stage('Getting Repo files') {
            steps {
                git branch: "main", credentialsId: 'jenkins-toknen', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${APP_NAME}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {

                        sh """
                            echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin

                            docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}

                            docker push ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                        """

                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            ls -l
                            sed 's|my-image|${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}|' k8s/deployment.yaml > deployment.yaml

                            kubectl apply -f deployment.yaml
                            kubectl apply -f k8s/service.yaml
                        """
                    }
                }
            }
        }
    }
}