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
            if (!fileExists('jenkins.properties')) {
                error "jenkins.properties not found"
            }
            def props = readProperties file: 'jenkins.properties'

            // Store into environment variables for later use
            env.GIT_REPO_URL = props['GIT_REPO_URL']
            env.BRANCH_NAME = props['BRANCH_NAME']
            env.IMAGE_NAME = props['IMAGE_NAME']
            env.IMAGE_TAG = props['IMAGE_TAG']
        }
    }
}


stage('Clone Repo') {
    steps {
        script {
            git credentialsId: 'github-cred-id', url: env.GIT_REPO_URL, branch: env.BRANCH_NAME
        }
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
