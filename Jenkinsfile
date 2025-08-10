pipeline {
    agent any
    environment {
        OWNER       = 'romanw05'
        IMAGE_NAME  = 'django_k3s_raspi_app'
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
        sh 'test -f django_mock_app/Dockerfile || { echo "Dockerfile missing at django_mock_app/Dockerfile"; exit 1; }'

        withCredentials([usernamePassword(credentialsId: 'ghcr', usernameVariable: 'GHCR_USER', passwordVariable: 'GHCR_PAT')]) {
          sh '''
            set -e
            export DOCKER_CONFIG="$(mktemp -d)"

            # Use COMMIT from Jenkins env if present, else derive from GIT_COMMIT
            COMMIT="${COMMIT:-$(echo "$GIT_COMMIT" | cut -c1-7)}"

            echo "$GHCR_PAT" | docker login ghcr.io -u "$GHCR_USER" --password-stdin

            docker build -f django_mock_app/Dockerfile \
              -t "${IMAGE}:${COMMIT}" \
              -t "${IMAGE}:latest" \
              django_mock_app

            docker push "${IMAGE}:${COMMIT}"
            docker push "${IMAGE}:latest"

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
