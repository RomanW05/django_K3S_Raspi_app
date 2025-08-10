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
        // sanity check: Dockerfile in subfolder
        sh 'test -f django_mock_app/Dockerfile || { echo "Dockerfile missing at django_mock_app/Dockerfile"; exit 1; }'

        withCredentials([usernamePassword(credentialsId: 'ghcr', usernameVariable: 'GHCR_USER', passwordVariable: 'GHCR_PAT')]) {
          sh '''
            set -e
            export DOCKER_CONFIG=$(mktemp -d)
            echo "$GHCR_PAT" | docker login ghcr.io -u "$GHCR_USER" --password-stdin

            # NOTE: context is the subfolder
            docker build -f django_mock_app/Dockerfile \
              -t ghcr.io/romanw05/django_mock_app:${GIT_COMMIT:0:7} \
              -t ghcr.io/romanw05/django_mock_app:latest \
              django_mock_app

            docker push ghcr.io/romanw05/django_mock_app:${GIT_COMMIT:0:7}
            docker push ghcr.io/romanw05/django_mock_app:latest
            docker logout ghcr.io || true
            rm -rf "$DOCKER_CONFIG"
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
