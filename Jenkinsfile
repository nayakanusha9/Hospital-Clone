pipeline {
    agent any 

    parameters {
        string(name: 'VERSION', defaultValue: 'v1.0.0', description: 'Enter the APP Version (e.g. v1.0.3, build-102)')
    }

    environment {
        DOCKERHUB_USER = "anushanayak091"         
        IMAGE_NAME = "hospital"               
        DOCKER_CREDS = "docker_creds"
    }

    stages {

        stage('Clone') {
            steps {
                echo "Cloning the GitHub Repository"
                git url: 'https://github.com/nayakanusha9/Hospital-Clone.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing dependencies..."
                sh "npm install || true"
            }
        }

        stage('Run Tests (Optional)') {
            steps {
                echo "Running tests..."
                sh "npm test || true"
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker Image..."
                sh """
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION} .
                docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker Image to Docker Hub"
                sh """
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION}
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Cleanup') {
            steps {
                echo "Cleaning up local images..."
                sh """
                docker rmi ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION} || true
                docker rmi ${DOCKERHUB_USER}/${IMAGE_NAME}:latest || true
                """
            }
        }
    }

    post {
        success {
            echo "✅ CI Pipeline SUCCESS - Image pushed successfully!"
        }
        failure {
            echo "❌ CI Pipeline FAILED"
        }
    }
}