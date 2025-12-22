pipeline {
    agent any

    environment {
        // VOTRE NOM D'UTILISATEUR DOCKER HUB
        DOCKER_USER = 'badr7788' 
        
        // L'ID de vos identifiants dans Jenkins (celui qu'on utilise depuis le début)
        DOCKER_CREDS_ID = 'dockerhub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                // Gets the code from your Git repo
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo '--- Building Images with Compose ---'
                    // Builds images as defined in docker-compose.yml
                    // Ajoutez --no-cache si vous voulez forcer la reconstruction
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
                    echo '--- Pushing Images ---'
                    // Attention: Cela ne marche que si les "image:" dans docker-compose.yml
                    // commencent par "badr7788/..."
                    sh 'docker-compose push'
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    echo '--- Deploying App ---'
                    // Stops old containers to free up ports
                    sh 'docker-compose down || true'
                    
                    // Starts the new containers in the background
                    // On ajoute --force-recreate pour être sûr de prendre la nouvelle image
                    sh 'docker-compose up -d --force-recreate'
                }
            }
        }
    }

    post {
        always {
            // Logout for security
            sh 'docker logout'
        }
        success {
            echo '✅ Build, Push, and Deploy Successful!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
