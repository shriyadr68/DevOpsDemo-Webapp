pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-web-app"
        CONTAINER_NAME = "webapp"
        PORT = "9090"
    }

    tools {
        // Use SonarQube Scanner configured in Jenkins
        sonarQubeScanner 'SonarScanner'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/shriyadr68/DevOpsDemo-Webapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=DevOpsDemo-Webapp \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://139.59.0.218:9000 \
                        -Dsonar.login=$SONARQUBE_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Cleanup Old Container') {
            steps {
                sh '''
                    if [ "$(docker ps -aq -f name=${CONTAINER_NAME})" ]; then
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true
                    fi
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh 'docker run -d -p ${PORT}:80 --name ${CONTAINER_NAME} ${IMAGE_NAME}:latest'
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