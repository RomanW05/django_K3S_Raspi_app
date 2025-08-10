pipeline {
    agent any
    environment {
        OWNER       = 'romanw05'
        IMAGE_NAME  = 'infra_images'
        REGISTRY    = 'ghcr.io'
        IMAGE       = "${REGISTRY}/${OWNER}/${IMAGE_NAME}"
        COMMIT      = "${env.GIT_COMMIT?.take(7)}"
        DEPLOY_DIR  = "/opt/django_k3s"
        REPO_URL    = "https://github.com/RomanW05/django_K3S_Raspi_app.git"
        BRANCH      = "main"
    }
  stages {
      stage('Checkout') {
        steps { 
            checkout scm
            sh 'echo "Checkout stage finished"'
             }
      }
      stage('Build & Push to GHCR') {
      when { branch 'main' }
      steps {
        withCredentials([usernamePassword(credentialsId: 'ghcr', usernameVariable: 'GHCR_USER', passwordVariable: 'GHCR_PAT')]) {
          sh '''
            set -e
            echo "$GHCR_PAT" | docker login ghcr.io -u "$GHCR_USER" --password-stdin
            docker build -t ${IMAGE}:${COMMIT} -t ${IMAGE}:latest .
            docker push ${IMAGE}:${COMMIT}
            docker push ${IMAGE}:latest
            docker logout ghcr.io
          '''
        }
      }
    }
  }
    post {
        success {
            echo "=== DEPLOYMENT SUCCESSFUL ==="
            echo "Files deployed to: ${DEPLOY_DIR}"
        }
        failure {
            echo "=== DEPLOYMENT FAILED ==="
            echo "Check the logs above for errors"
        }
    }
}
