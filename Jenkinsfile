pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'julienvb/datascientest'
        DOCKER_TAG_CAST_SERVICE = 'service-qa'
        DOCKER_TAG_MOVIE_SERVICE = 'service-qa'
        DOCKER_USERNAME = 'julienvb'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'qa', url: 'https://github.com/JulienVbn/evaluation-jenkins.git'
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

        stage('Deploy for qa') {
            steps {
                script {
                sh '''
                sed -i 's|julienvb/datascientest:movie-service|&-qa|' iac/values.yaml
                sed -i 's|julienvb/datascientest:cast-service|&-qa|' iac/values.yaml
                helm upgrade --install -f iac/values.yaml -f iac/environments/values.qa.yaml datascientest-evaluation-qa iac/ --kubeconfig /etc/rancher/k3s/k3s.yaml
                '''
                }
            }
        }
    }
}