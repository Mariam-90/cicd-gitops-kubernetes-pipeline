pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'DIRECT_DEPLOY',
            defaultValue: true,
            description: 'Also kubectl-deploy directly (mandatory requirement). Disable once Argo CD owns deployment.'
        )
    }

    environment {
        DOCKERHUB_USER = 'maryam19455'
        IMAGE_NAME = "${DOCKERHUB_USER}/todo-app"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        GIT_REPO_URL = 'https://github.com/Mariam-90/cicd-gitops-kubernetes-pipeline.git'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${GIT_REPO_URL}"
            }
        }
    }
}