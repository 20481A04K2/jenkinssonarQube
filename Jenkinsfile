pipeline {
  agent any

  environment {
    PROJECT_ID = 'sylvan-hydra-464904-d9'
    REGION = 'us-central1'
    REPOSITORY = 'flask-app-repo'
    IMAGE_NAME = 'flask-app'
    IMAGE_TAG = 'latest'
    AR_URL = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $AR_URL .'
      }
    }

    stage('Push to Artifact Registry') {
      steps {
        sh '''
          gcloud config set project $PROJECT_ID
          gcloud auth configure-docker $REGION-docker.pkg.dev --quiet
          docker push $AR_URL
        '''
      }
    }

    stage('Run Docker Image') {
      steps {
        sh '''
          docker stop flask-container || true
          docker rm flask-container || true
          docker pull $AR_URL
          docker run -d -p 5000:5000 --name flask-container $AR_URL
        '''
      }
    }
  }
}
