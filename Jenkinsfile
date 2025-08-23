pipeline {
    agent any
    environment {
        OWNER       = 'romanw05'
        IMAGE_NAME  = 'django_k3s_raspi_app'
        REGISTRY    = 'ghcr.io'
        IMAGE       = "${REGISTRY}/${OWNER}/${IMAGE_NAME}"
        COMMIT      = "${env.GIT_COMMIT?.take(7)}"
        DEPLOY_DIR  = "/opt/django_k3s"
        REPO_URL    = "git@github.com:RomanW05/django_K3S_Raspi_app.git"
        BRANCH      = "main"

        NAMESPACE = 'apps'
        RELEASE   = 'django-mock-app'
        CHART_DIR = 'helm/django-mock-app'
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
          sh '''#!/usr/bin/env bash
          set -euo pipefail
          export DOCKER_CONFIG="$(mktemp -d)"
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
    stage('Deploy to K3s (Helm)') {
      when { branch 'main' }
      steps {
        sh '''#!/usr/bin/env bash
        set -euo pipefail

        # Compute short SHA safely
        COMMIT="${COMMIT:-$(echo "$GIT_COMMIT" | cut -c1-7)}"

        # Sanity checks
        command -v kubectl >/dev/null
        command -v helm    >/dev/null || { echo "Helm not found. Install Helm v3 on the agent."; exit 127; }

        # Confirm kube access (jenkins kubeconfig is set up)
        kubectl get ns >/dev/null

        # Deploy / upgrade
        helm upgrade --install "${RELEASE}" "${CHART_DIR}" \
          --namespace "${NAMESPACE}" --create-namespace \
          --set image.repository="${IMAGE}" \
          --set image.tag="${COMMIT}" \
          --set-string imagePullSecrets[0].name=ghcr \
          --wait --timeout 5m

        # Rollout confirmation (redundant when using --wait, but nice in logs)
        kubectl -n "${NAMESPACE}" rollout status deploy/${RELEASE}
        '''
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
