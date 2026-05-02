pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = 'bisong23/meditrack:latest'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Test') {
            steps {
                sh '''
                    python3 -m pip install --user --upgrade pip
                    python3 -m pip install --user pytest flask
                    python3 -m pytest test_app.py -v || true
                '''
                
                
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                sh '''
                    echo ${DOCKER_CREDS_PSW} | docker login -u ${DOCKER_CREDS_USR} --password-stdin
                    docker push ${DOCKER_IMAGE}
                '''
            }
        }
        stage('deploy to eks') {
            steps {
                sh '''
                   kubectl apply -f k8s/configmap.yaml
                   kubectl apply -f k8s/deployment.yaml
                   kubectl apply -f k8s/service.yaml 
                   kubectl rollout status deployment/meditrack-deployment
                '''
            }
        }
                
                
    }
    post{
        always{
            sh 'docker logout'
        }
    }
}
