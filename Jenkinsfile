pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'julienvb/datascientest'
        DOCKER_TAG_CAST_SERVICE = 'cast-service-staging'
        DOCKER_TAG_MOVIE_SERVICE = 'movie-service-staging'
        DOCKER_USERNAME = 'julienvb'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'staging', url: 'https://github.com/JulienVbn/evaluation-jenkins.git'
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

        stage('Deploy for staging') {
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                sed -i 's|julienvb/datascientest:movie-service|&-staging|' iac/values.yaml
                sed -i 's|julienvb/datascientest:cast-service|&-staging|' iac/values.yaml
                helm upgrade --install -f iac/values.yaml -f iac/environments/values.staging.yaml datascientest-evaluation-staging iac/ --kubeconfig .kube/config
                '''
                }
            }
        }
    }
}