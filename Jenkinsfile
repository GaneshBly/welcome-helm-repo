pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ganeshopen/welcome-js-app"
        IMAGE_TAG = "latest"
        HELM_CHART_PATH = "./helm/my-js-app-helm-chart"
        K8S_CLUSTER = "docker-desktop"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'docker-credentials') {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
        stage('Helm Install or Upgrade') {
            steps {
                script {
                    sh """
                        helm repo add my-helm-repo https://github.com/GaneshBly/welcome-helm-repo
                        helm repo update
                        helm upgrade --install welcome-helm-repo ./my-js-app-helm-chart --namespace default --set image.repository=${DOCKER_IMAGE} --set image.tag=${IMAGE_TAG}
                    """
                }
            }
        }
        stage('Package Helm Chart') {
            steps {
                script {
                    sh """
                        helm package ${HELM_CHART_PATH}
                    """
                }
            }
        }
        stage('Upload Helm Chart to Repo') {
            steps {
                script {
                    sh """
                        helm repo add welcome-helm-repo https://github.com/GaneshBly/welcome-helm-repo
                        helm repo update
                        helm package ${HELM_CHART_PATH}
                        mv *.tgz charts/
                        helm repo index . --url https://github.com/GaneshBly/welcome-helm-repo
                        git add .
                        git commit -m "Add updated Helm chart"
                        git push origin main
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}