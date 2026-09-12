pipeline {
    agent any
    
    environment {
        DOCKERHUB_CRED = credentials('dockerhub-creds')
        IMAGE_NAME = 'abdelhayyaser/backend-go'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AbdulhayYassir/go-ci-dc.git'
            }
        }
        
        stage('Build & Test Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CRED_PSW | docker login -u $DOCKERHUB_CRED_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh """
                        git config user.email "abdulhayyassir@gmail.com"
                        git config user.name "AbdulhayYassir"
                        
                        if [ -f K8S/backend_deployment.yaml ]; then
                            sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" K8S/backend_deployment.yaml
                            git add K8S/backend_deployment.yaml
                        elif [ -f k8s/deployment.yaml ]; then
                            sed -i "s|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" k8s/deployment.yaml
                            git add k8s/deployment.yaml
                        fi
                        
                        if ! git diff --staged --quiet; then
                            git commit -m "Update image tag to ${IMAGE_TAG} [skip ci]"
                            git push https://${GIT_TOKEN}@github.com/AbdulhayYassir/go-ci-dc.git HEAD:main
                        else
                            echo "No changes to commit."
                        fi
                    """
                }
            }
        }
    }
}
