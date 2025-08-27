pipeline {
    agent any
    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "rahulrk410/demo-app"
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
                    COMMIT = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    IMAGE_TAG = "${REGISTRY}/${IMAGE_NAME}:${COMMIT}"
                    env.IMAGE_TAG = IMAGE_TAG
                }
                sh 'docker build -t $IMAGE_TAG .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker push $IMAGE_TAG
                        docker tag $IMAGE_TAG ${REGISTRY}/${IMAGE_NAME}:latest
                        docker push ${REGISTRY}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh '''
                    docker rm -f demo-app || true
                    docker run -d --name demo-app -p 80:80 $IMAGE_TAG
                '''
            }
        }
    }
    post {
        success {
            echo "✅ Deployment successful"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}
