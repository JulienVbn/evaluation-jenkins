pipeline {
    agent any
    
    environment {
        DOCKER_ID = 'julienvb'
        DOCKER_IMAGE = 'datascientest'
        DOCKER_TAG_CAST_SERVICE = "cast-service-prod.v${BUILD_ID}.0"
        DOCKER_TAG_MOVIE_SERVICE = "movie-service-prod.v${BUILD_ID}.0"
        DOCKER_USERNAME = 'julienvb'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'Master', url: 'https://github.com/JulienVbn/evaluation-jenkins.git'
            }
        }
        
        stage('Build Docker Image - Cast Service') {
            steps {
                script {
                    sh '''
                    cd cast-service
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG_CAST_SERVICE .
                    '''
                }
            }
        }

        stage('Build Docker Image - Movie Service') {
            steps {
                script {
                    sh '''
                    cd movie-service
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG_MOVIE_SERVICE .
                    '''
                }
            }
        }    
        
        stage('Push Docker Image') {
            steps {
                withCredentials([
                    string(credentialsId: 'docker_hub_password', variable: 'DOCKER_PASSWORD')
                ])
                {
                    sh '''
                    echo "$DOCKER_PASSWORD" > credentials.txt
                    cat credentials.txt | docker login -u $DOCKER_USERNAME --password-stdin
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG_CAST_SERVICE
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG_MOVIE_SERVICE
                    rm credentials.txt
                    '''
                }
            }
        }

        stage('Deploy for production') {
            environment {
                KUBECONFIG = credentials("kubeconfig")
            }
            steps {
                    timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    sed -i "s|${DOCKER_ID}\\/${DOCKER_IMAGE}:movie-service|${DOCKER_ID}\\/${DOCKER_IMAGE}:${DOCKER_TAG_MOVIE_SERVICE}|" iac/values.yaml
                    sed -i "/^${DOCKER_ID}\\/${DOCKER_IMAGE}:cast-service/s/.*/${DOCKER_ID}\\/${DOCKER_IMAGE}:${DOCKER_TAG_CAST_SERVICE}/" iac/values.yaml
                    cat iac/values.yaml
                    helm upgrade --install -f iac/values.yaml -f iac/environments/values.prod.yaml datascientest-evaluation-prod iac/ --kubeconfig .kube/config
                    '''
                }
            }
        }
    }
}