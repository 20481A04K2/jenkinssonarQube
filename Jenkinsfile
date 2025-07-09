pipeline {
  agent any

  environment {
    PROJECT_ID = 'sylvan-hydra-464904-d9'
    REGION = 'us-central1'
    REPOSITORY = 'flask-app-repo'
    IMAGE_NAME = 'flask-app'
    IMAGE_TAG = 'latest'
    AR_URL = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}"
    SONAR_SCANNER_HOME = tool 'Default Sonar Scanner'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
          withSonarQubeEnv('My SonarQube Server') {
            sh """
              ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                -Dsonar.projectKey=flask-app \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://34.63.76.155:9000 \
                -Dsonar.login=$SONAR_TOKEN
            """
          }
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $AR_URL .'
      }
    }

    stage('Push to Artifact Registry') {
      steps {
        withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
            gcloud config set project $PROJECT_ID
            gcloud auth configure-docker $REGION-docker.pkg.dev --quiet
            docker push $AR_URL
          '''
        }
      }
    }

    stage('Run Docker Image on Jenkins VM') {
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
