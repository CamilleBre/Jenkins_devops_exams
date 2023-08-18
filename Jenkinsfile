pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "camillebre" // replace this with your docker-id
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
        
    }
    agent any // Jenkins will be able to select all available agents
    stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build --platform=linux/amd64 -t $DOCKER_ID/movie_service:$DOCKER_TAG ./movie-service
                 docker build --platform=linux/amd64 -t $DOCKER_ID/cast_service:$DOCKER_TAG ./cast-service
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

        stage('Deploy with helm on dev') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'eu-west-3') {
                sh '''
                rm -Rf ~/.kube
                aws eks update-kubeconfig --name my-eks
                helm upgrade --install --set  docker_tag=$DOCKER_TAG dev-chart ./helm --values=./helm/values.yaml --kubeconfig ~/.kube/config --namespace dev --create-namespace
                
                echo 'Helm deployed on dev'
                '''
                } 
            }
        }
        stage('Deploy with helm on qa') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'eu-west-3') {
                sh '''
                rm -Rf ~/.kube
                aws eks update-kubeconfig --name my-eks 

                helm upgrade --install --set  docker_tag=$DOCKER_TAG qa-chart ./helm --values=./helm/values.yaml --kubeconfig ~/.kube/config --namespace qa --create-namespace
                
                echo 'Helm deployed on qa'
                '''
                } 
            }
        }

        stage('Deploy with helm on staging') {    
            steps {
                withAWS(credentials: 'aws-credentials', region: 'eu-west-3') {
                sh '''
                rm -Rf ~/.kube
                aws eks update-kubeconfig --name my-eks 

                helm upgrade --install --set  docker_tag=$DOCKER_TAG staging-chart ./helm --values=./helm/values.yaml --kubeconfig ~/.kube/config --namespace staging --create-namespace
                
                echo 'Helm deployed on staging'
                '''
                } 
            }

        }
        stage('Deploy with helm on prod') {
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                withAWS(credentials: 'aws-credentials', region: 'eu-west-3') {
                sh '''
                rm -Rf ~/.kube
                aws eks update-kubeconfig --name my-eks

                helm upgrade --install --set  docker_tag=$DOCKER_TAG prod-chart ./helm --values=./helm/values.yaml --kubeconfig ~/.kube/config --namespace prod --create-namespace
                
                echo 'Helm deployed on prod'
                '''
                } 
            }
        }   
    }
}

