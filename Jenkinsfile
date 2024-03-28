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

        stage('Deploy for production') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    script {
                        def kubeconfigPath = "${env.WORKSPACE}/kubeconfig.yml"
                        sh "cp $KUBECONFIG_FILE $kubeconfigPath"
                        def chartName = 'datascientest-evaluation-prod'
                        def chartExists = sh(returnStdout: true, script: "helm list -q --kubeconfig $kubeconfigPath | grep -q '^$chartName' && echo 'true' || echo 'false'").trim()
                        if (chartExists == 'true') {
                            echo "Mise à jour du chart existant..."
                            sh "helm uninstall $chartName --kubeconfig $kubeconfigPath"
                            sh "helm install -f iac/values.yaml -f iac/environments/values.prod.yaml $chartName iac/ --kubeconfig $kubeconfigPath"
                        } else {
                            echo "Application du nouveau chart..."
                            sh "helm install -f iac/values.yaml -f iac/environments/values.prod.yaml $chartName iac/ --kubeconfig $kubeconfigPath"
                        }
                    }
                }
            }
        }
    }
}