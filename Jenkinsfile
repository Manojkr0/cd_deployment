pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_NAME', defaultValue: 'manojkumar8008/myapp1', description: 'Docker Image Name (e.g., user/image)')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag')
        string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Git Branch to Checkout')
        string(name: 'GIT_REPO_URL', defaultValue: 'https://github.com/Manojkr0/python_code.git', description: 'Git Repository URL')
    }

    environment {
        DOCKER_IMAGE = "${params.IMAGE_NAME}:${params.IMAGE_TAG}"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: "${params.BRANCH_NAME}", credentialsId: 'github-cred-id', url: "${params.GIT_REPO_URL}"
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
            echo "✅ Docker image pushed: $DOCKER_IMAGE"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
