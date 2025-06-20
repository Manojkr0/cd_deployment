pipeline {
    agent any

    parameters {
        string(name: 'GIT_REPO_URL', description: 'Git repository URL to clone')
        string(name: 'BRANCH_NAME', description: 'Branch name to build')
    }



    stages {
        stage('Clone Repo') {
            steps {
                script {
                    git credentialsId: 'github-cred-id',
                        url: "${params.GIT_REPO_URL}",
                        branch: "${params.BRANCH_NAME}"
                }
            }
        }

        stage('Read Properties') {
            steps {
                script {
                    def props = readProperties file: 'jenkins.properties'
                    env.IMAGE_NAME = props['IMAGE_NAME']
                    env.IMAGE_TAG  = props['IMAGE_TAG']
                    env.DOCKER_IMAGE = "${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                    echo "📦 Image to build: ${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    script {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push ${env.IMAGE_NAME}:${env.IMAGE_TAG}
                            docker logout
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed: ${env.IMAGE_NAME}:${env.IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
