pipeline {
    agent any
    
    environment {
        DOCKERHUB_CRED = credentials('dockerhub-creds') // ID Jenkins
        IMAGE_NAME = 'abdelhayyaser/backend-go' // DockerHub
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
                sh """
                    sed -i 's|image: .*backend.*|image: $IMAGE_NAME:$IMAGE_TAG|g' K8S/backend_deployment.yaml
                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins CI"
                    git add K8S/backend_deployment.yaml
                    git commit -m "Update backend image tag to $IMAGE_TAG [skip ci]" || echo "No changes to commit"
                    git push origin main
                """
            }
        }
    }
}
