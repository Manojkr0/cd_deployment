pipeline {
    agent any

    environment {
        // Define empty, will populate in script block
        GIT_REPO_URL = ''
        IMAGE_NAME = ''
        IMAGE_TAG = ''
        BRANCH_NAME = ''
    }

    stages {
        stage('Read Properties') {
            steps {
                script {
                    def propsFile = 'jenkins.properties'
                    if (!fileExists(propsFile)) {
                        error "❌ File '${propsFile}' not found in workspace!"
                    }

                    def props = readProperties file: propsFile

                    env.GIT_REPO_URL = props['GIT_REPO_URL']
                    env.IMAGE_NAME = props['IMAGE_NAME']
                    env.IMAGE_TAG = props['IMAGE_TAG']
                    env.BRANCH_NAME = props['BRANCH_NAME']
                    env.DOCKER_IMAGE = "${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Clone Repo') {
            steps {
                git branch: "${env.BRANCH_NAME}", credentialsId: 'github-cred-id', url: "${env.GIT_REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed: ${env.DOCKER_IMAGE}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
