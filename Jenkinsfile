pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'julienvb/datascientest'
        DOCKER_TAG_CAST_SERVICE = 'cast-service-prod'
        DOCKER_TAG_MOVIE_SERVICE = 'movie-service-prod'
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
                    docker build -t $DOCKER_IMAGE:$DOCKER_TAG_CAST_SERVICE .
                    '''
                }
            }
        }

        stage('Build Docker Image - Movie Service') {
            steps {
                script {
                    sh '''
                    cd movie-service
                    docker build -t $DOCKER_IMAGE:$DOCKER_TAG_MOVIE_SERVICE .
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
                    docker push $DOCKER_IMAGE:$DOCKER_TAG_CAST_SERVICE
                    docker push $DOCKER_IMAGE:$DOCKER_TAG_MOVIE_SERVICE
                    rm credentials.txt
                    '''
                }
            }
        }

        stage('Deploy for production') {
            steps {
                    timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }
                script {
                    sh '''
                    sed -i 's|julienvb/datascientest:movie-service|&-prod|' iac/values.yaml
                    sed -i 's|julienvb/datascientest:cast-service|&-prod|' iac/values.yaml
                    helm uninstall datascientest-evaluation-prod --kubeconfig /etc/rancher/k3s/k3s.yaml
                    helm upgrade --install -f iac/values.yaml -f iac/environments/values.prod.yaml datascientest-evaluation-prod iac/ --kubeconfig /etc/rancher/k3s/k3s.yaml
                    '''
                }
            }
        }
    }
}