@Library('kmenzli-shared-lib')_
pipeline {
    agent any
        environment {
            IMAGE_NAME = 'webapp'
            IMAGE_VERSION = 'v1'
            DOCKER_PASSWORD = credentials ('docker-password')
            DOCKER_USERNAME = 'kmenzli'
            HOST_PORT = 80
            DOCER_PORT = 80
            IP_DOCKER = 'localhost'
        }

    stages {   
        stage('Build') {
            steps {
                echo 'Building..'
                steps {
                    script {
                        sh '''
                            docker --no-cache tag $IMAGE_NAME:$IMAGE_VERSION
                        '''
                    }
                 }
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
                steps {
                    script {
                        sh '''
                            docker run --rm -dp $HOST_PORT:$DOCKER_PORT --name $IMAGE_NAME $IMAGE_NAME:$IMAGE_VERSION
                            slep 5
                            curl -I http://$IP_DOCKER
                            sleep 5 
                            docker stop --name $IMAGE_NAME
                        '''
                    }
                 }
            }
        }
        stage('Release') {
            steps {
                echo 'Deploying....'
                steps {
                    script {
                        sh '''
                        docker tag $IMAGE_NAME:$IMAGE_VERSION $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        docker push $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION
                        '''
                    }
                 }
            }
        }
         stage('Deploy Preview ') {
            environment {
                SERVER_IP = '35.180.86.203' 
                SERVEUR_USERNAME = 'kmenzli'
            }
            steps {
                
                steps {
                    script {
                        timeout(time: 30, unit: "MINUTES") {
                            input message : "Voulez vous rélaiser un déploiemnt sur l'environement Preview? ", ok : 'YES'
                        }
                        sshagent(['key-pair']) {
                        sh '''
                         echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker rm -f $IMAGE_NAME || echo 'All deleted' "
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION || echo 'Image dowlnoad successfully' "
                         sleep 30 
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$DOCKER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION "
                        sleep 5 
                         curl -I http://$SERVER_IP:$HOST_PORT

                        '''
                        }
                       
                    }
                 }
            }
        }
         stage('Deploy Staging ') {
            
            environment {
                SERVER_IP = '35.180.189.201'
                SERVEUR_USERNAME =  'kmenzli'
            }
            steps {
                echo 'Deploying....'
                steps {
                    script {
                         timeout(time: 30, unit: "MINUTES") {
                            input message : "Voulez vous rélaiser un déploiemnt sur l'environement Staging? ", ok : 'YES'
                        }
                        sshagent(['key-pair']) {
                        sh '''
                         echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker rm -f $IMAGE_NAME || echo 'All deleted' "
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION || echo 'Image dowlnoad successfully' "
                         sleep 30 
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$DOCKER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION "
                        sleep 5 
                         curl -I http://$SERVER_IP:$HOST_PORT

                        '''
                        }
                       
                    }
                 }
            }
        }
         stage('Deploy Prod ') {
            when {expression {GIT_BRANCH == 'origin/prod'}}
            environment {
                SERVER_IP = '35.180.39.149' 
                SERVEUR_USERNAME =  'kmenzli'
            }
            steps {
                echo 'Deploying....'
                steps {
                    script {
                         timeout(time: 30, unit: "MINUTES") {
                            input message : "Voulez vous rélaiser un déploiemnt sur l'environement PROD? ", ok : 'YES'
                        }
                        sshagent(['key-pair']) {
                        sh '''
                         echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker rm -f $IMAGE_NAME || echo 'All deleted' "
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION || echo 'Image dowlnoad successfully' "
                         sleep 30 
                         ssh -o StricHostKeyChecking=o -l $SERVEUR_USERNAME $SERVER_IP "docker run --rm -dp $HOST_PORT:$DOCKER_PORT --name $IMAGE_NAME $DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_VERSION "
                        sleep 5 
                         curl -I http://$SERVER_IP:$HOST_PORT

                        '''
                        }
                       
                    }
                 }
            }
        }
        post {
            always {             
                
                    script {
                       slackNotifier currentBuild.result
                    }
            }
        }
    }
}
