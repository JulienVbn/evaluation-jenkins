pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'julienvb/labo'
        DOCKER_TAG = 'datascientest-project-cast-service'
        DOCKER_CREDENTIALS_ID = 'julien-credentials'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/JulienVbn/evaluation-jenkins.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        cd cast-service
                        docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                    '''
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }
    }
}