pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "camillebre" // replace this with your docker-id
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
        NAMESPACE = "dev"
        AWS_CREDENTIALS = credentials('aws-credentials')
    }
    agent any // Jenkins will be able to select all available agents

    stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/movie_service:$DOCKER_TAG ./movie-service
                 docker build -t $DOCKER_ID/cast_service:$DOCKER_TAG ./cast-service
                 sleep 6
                '''
                }
            }
        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/movie_service:$DOCKER_TAG
                docker push $DOCKER_ID/cast_service:$DOCKER_TAG
                '''
                }
            }

        }

        stage('Create namespace') {
            
            steps {
                sh '''
                rm -Rf ~/.aws
                mkdir ~/.aws
                cat $AWS_CREDENTIALS > ~/.aws/credentials

                #rm -Rf ~/.kube 
                #mkdir ~/.kube
                #cat $KUBE_CONFIG > ~/.kube/config
                aws eks update-kubeconfig --name my_eks
                    
                #rm -Rf ~/.aws
                #mkdir ~/.aws
                #cat $CREDENTIAL > ~/.aws/credentials
                #ls 
                #cat ~/.kube/config
                #cat ~/.aws/credentials
                #kubectl get nodes
                #kubectl apply -f  role.yaml
                
                kubectl create namespace $NAMESPACE
                
                echo 'namespace created'
                '''
            }

        }

    }   
}

