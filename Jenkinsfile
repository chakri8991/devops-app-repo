pipeline {
    agent any

    environment {
        IMAGE_NAME = "chakri8991/devopsecom"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    options {
        skipDefaultCheckout()
    }

    stages {

        stage('Checkout Repo') {
            steps {
                git branch: 'main',
                url: 'https://github.com/chakri8991/devopsecom1.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('devops-app-repo') {
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'PASS')]) {
                    sh '''
                    docker login -u chakri8991 -p $PASS
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                sh '''
                sed -i "s|image:.*|image: $IMAGE_NAME:$IMAGE_TAG|" \
                devops-gitops-repo/k8s-manifests/deployment.yaml
                '''
            }
        }

        stage('Commit & Push GitOps Changes') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    git config user.name "jenkins"
                    git config user.email "jenkins@ci.com"
                    git add devops-gitops-repo/k8s-manifests/deployment.yaml
                    git commit -m "Updated image to $IMAGE_TAG" || true
                    git push https://$USER:$PASS@github.com/chakri8991/devopsecom1.git main
                    '''
                }
            }
        }
    }
}
