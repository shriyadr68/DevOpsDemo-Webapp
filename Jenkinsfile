pipeline {
    agent any
    environment {
        IMAGE_NAME = "my-web-app"
        CONTAINER_NAME = "webapp"
        PORT = "8081"
    }
    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/shriyadr68/DevOpsDemo-Docker.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }
        stage('Cleanup Old Container') {
            steps {
                sh '''
                if [ "$(docker ps -q -f name=${CONTAINER_NAME})" ]; then
                    docker stop ${CONTAINER_NAME}
                    docker rm ${CONTAINER_NAME}
                fi
                '''
            }
        }
        stage('Run New Container') {
            steps {
                sh 'docker run -d -p ${PORT}:80 --name ${CONTAINER_NAME} ${IMAGE_NAME}'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker tag ${IMAGE_NAME}:latest ${DOCKER_USER}/${IMAGE_NAME}:latest
                    docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }
    }
}
