pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'sapkotp2-dockerhub'
        DOCKER_IMAGE = 'cithit/sapkotp2'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/parbatisapkota/225-lab3-3.git'
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }
        stage('Lint HTML') {
            steps {
                sh 'npm install htmlhint --save-dev'
                sh 'npx htmlhint *.html'
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
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
        stage('Deploy to Dev Environment') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'sapkotp2-225', variable: 'KUBECONFIG')]) {
                        sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                        sh "kubectl apply -f deployment-dev.yaml"
                    }
                }
            }
        }
        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'sapkotp2-225', variable: 'KUBECONFIG')]) {
                        sh "kubectl get all"
                    }
                }
            }
        }
    }
    post {
        success {
            slackSend(channel: '#builds', color: "good",
                      message: "Build SUCCESS: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
        }
        unstable {
            slackSend(channel: '#builds', color: "warning",
                      message: "Build UNSTABLE: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
        }
        failure {
            slackSend(channel: '#builds', color: "danger",
                      message: "Build FAILED: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
        }
    }
}