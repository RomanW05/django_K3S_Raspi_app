pipeline {
    agent any
    environment {
        DEPLOY_DIR = "/opt/django_k3s"
        REPO_URL = "https://github.com/RomanW05/django_K3S_Raspi_app.git"
        BRANCH = "main"
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
