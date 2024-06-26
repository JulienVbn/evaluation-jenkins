pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'julienvb/labo'
        DOCKER_TAG_CAST_SERVICE = 'datascientest-project-cast-service'
        DOCKER_TAG_MOVIE_SERVICE = 'datascientest-project-movie-service'
        DOCKER_USERNAME = 'julienvb'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/JulienVbn/evaluation-jenkins.git'
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
                helm upgrade --install -f iac/values.yaml -f iac/environments/values.qa.yaml datascientest-evaluation-qa iac/ --kubeconfig /etc/rancher/k3s/k3s.yaml
                '''
                }
            }
        }

        stage('Deploy for staging') {
            steps {
                script {
                sh '''
                helm upgrade --install -f iac/values.yaml -f iac/environments/values.staging.yaml datascientest-evaluation-staging iac/ --kubeconfig /etc/rancher/k3s/k3s.yaml
                '''
                }
            }
        }

        stage('Deploy for dev') {
            steps {
                script {
                sh '''
                helm upgrade --install -f iac/values.yaml -f iac/environments/values.dev.yaml datascientest-evaluation-dev iac/ --kubeconfig /etc/rancher/k3s/k3s.yaml
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
                helm upgrade --install -f iac/values.yaml -f iac/environments/values.prod.yaml datascientest-evaluation-prod iac/ --kubeconfig /etc/rancher/k3s/k3s.yaml
                '''
                }
            }
        }
    }
}