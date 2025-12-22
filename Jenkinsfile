pipeline {
    agent any

    environment {
        DOCKER_USER = 'badr7788' 
        DOCKER_CREDS_ID = 'dockerhub-credentials'
        
        // On définit les noms ici pour éviter les erreurs de frappe
        IMAGE_BACKEND = "${DOCKER_USER}/todo-backend"
        IMAGE_FRONTEND = "${DOCKER_USER}/todo-frontend"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo '--- Building Images with Compose ---'
                    // Construit la version 'latest' définie dans docker-compose.yml
                    sh 'docker-compose build'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    echo '--- Logging in ---'
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDS_ID, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    }
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    echo '--- Pushing "latest" Image ---'
                    // 1. Pousse la version 'latest' (pour la prod)
                    sh 'docker-compose push'
                    
                    echo "--- Tagging and Pushing Version : ${BUILD_NUMBER} ---"
                    // 2. On crée manuellement le tag avec le numéro de build
                    sh "docker tag ${IMAGE_BACKEND}:latest ${IMAGE_BACKEND}:${BUILD_NUMBER}"
                    sh "docker tag ${IMAGE_FRONTEND}:latest ${IMAGE_FRONTEND}:${BUILD_NUMBER}"
                    
                    // 3. On pousse ces versions spécifiques
                    sh "docker push ${IMAGE_BACKEND}:${BUILD_NUMBER}"
                    sh "docker push ${IMAGE_FRONTEND}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    echo '--- Deploying App ---'
                    sh 'docker-compose down || true'
                    sh 'docker-compose up -d --force-recreate'
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
        success {
            echo "✅ Success! Version ${BUILD_NUMBER} is live."
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
