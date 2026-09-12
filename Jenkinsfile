pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-id') // ID الحساب في جينكنز
        IMAGE_NAME = 'abdelhayyaser/go-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        GIT_MANIFEST_REPO = 'https://github.com/AbdulhayYassir/go-ci-dc.git' // أو repo الـ manifests
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AbdulhayYassir/go-ci-dc.git'
            }
        }
        
        stage('Test & Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }
        
        stage('Push Image to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }
        
        stage('Update K8s Manifest Tag') {
            steps {
                sh """
                    sed -i 's|image: $IMAGE_NAME:.*|image: $IMAGE_NAME:$IMAGE_TAG|g' k8s/deployment.yaml
                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins CI"
                    git add k8s/deployment.yaml
                    git commit -m "Update image tag to $IMAGE_TAG [skip ci]"
                    git push origin main
                """
            }
        }
    }
}
