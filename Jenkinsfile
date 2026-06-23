pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'DIRECT_DEPLOY',
            defaultValue: true,
            description: 'Also kubectl-deploy directly. Disable once Argo CD owns deployment.'
        )
    }

    environment {
        DOCKERHUB_USER = 'maryam19455'
        IMAGE_NAME     = "${DOCKERHUB_USER}/todo-app"
        IMAGE_TAG      = "v${BUILD_NUMBER}"
        GIT_REPO_URL   = 'https://github.com/Mariam-90/cicd-gitops-kubernetes-pipeline.git'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${GIT_REPO_URL}"
            }
        }

        stage('Build Application') {
            steps {
                dir('app') {
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --no-cache-dir -r requirements.txt
                        python -m py_compile app.py
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest \
                        .
                '''
            }
        }

        stage('Security Scan - Trivy') {
            steps {
                sh '''
                    trivy image \
                        --exit-code 0 \
                        --severity LOW,MEDIUM,HIGH,CRITICAL \
                        ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    stage('Deploy to Kubernetes (direct / mandatory requirement)') {
            when {
                expression { return params.DIRECT_DEPLOY }
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        kubectl set image deployment/todo-app todo-app=${IMAGE_NAME}:${IMAGE_TAG}
                        kubectl rollout status deployment/todo-app --timeout=120s
                    '''
                }
            }
        }
    

    post {
        success {
            echo "Pipeline succeeded: ${IMAGE_NAME}:${IMAGE_TAG} built, scanned, and pushed."
        }

        failure {
            echo "Pipeline failed. Check the Console Output."
        }
    }
}