pipeline {
    agent any

    environment {
        IMAGE_NAME = "vishum07/django-app"
        IMAGE_TAG = "latest"
        CONTAINER = "django-container"
    }
    stages {
        stage(checkout) {
            steps {
                checkout scm
            }
        }
        stage ('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME:$IMAGE_TAG ."
            }
        }
        stage ('Push To Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                    )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push $IMAGE_NAME:$IMAGE_TAG" 
                }
            }
        }
        stage ('Deploy to EC2') {
          steps {
            sh '''
                echo "Deploying to EC2..."
                echo "Pulling latest image from Docker Hub..."
                docker pull $IMAGE_NAME:$IMAGE_TAG

                echo "Stopping existing container (if any)..."
                docker stop $CONTAINER || true

                echo "Removing existing container (if any)..."
                docker rm $CONTAINER || true

                echo "Running new container..."
                docker run -d --name $CONTAINER -p 8000:8000 $IMAGE_NAME:$IMAGE_TAG
            '''
          }
        }
        stage ('Deploy to Kubernetes') {
          steps {
            sh '''
                echo "Deploying to Kubernetes..."
                echo "Updating Kubernetes deployment with new image..."
                kubectl apply -f k8s/
                kubectl set image deployment/django-deployment django=$IMAGE_NAME:$IMAGE_TAG
            '''
          }
        }
    }
}
