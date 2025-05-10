pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'f84d68c0-d4b6-461e-8890-bd5e45e5cb37'  // Docker Hub credentials ID
        IMAGE_NAME = 'spandu6677/cicd-project'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/spandana-lab/cicd-project.git', branch: 'main'
            }
        }
        stage('Check Docker') {
            steps {
                script {
                    sh 'docker --version'
                    sh 'docker info'
                }
            }
        }
        stage('Verify Dockerfile') {
            steps {
                script {
                    sh 'ls -l Dockerfile'
                }
            }
        }
        stage('SonarQube Analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                    args '--network host --entrypoint="" -v ${WORKSPACE}:${WORKSPACE}:rw,z'
                }
            }
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('mysonar') {
                        sh '''
                            export SONAR_USER_HOME=${WORKSPACE}/.sonar
                            sonar-scanner \
                                -Dsonar.projectKey=cicd-project \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://54.203.99.36:9000 \
                                -Dsonar.scm.provider=git \
                                -Dsonar.ws.timeout=120 \
                                -Dsonar.inclusions=**/*.js,**/*.ts
                            ls -l ${SONAR_USER_HOME} || echo "Sonar cache directory not found"
                            ls -l .scannerwork || echo "Scanner work directory not found"
                        '''
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        def image = docker.image("${IMAGE_NAME}:${IMAGE_TAG}")
                        image.push()
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                        image.push('latest')
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([string(credentialsId: 'jwt-secret', variable: 'JWT_SECRET')]) {
                    sh """
                        # Create a Docker network for the app and MongoDB
                        docker network create auth-network || true

                        # Stop and remove existing containers
                        docker stop user-authentication-app || true
                        docker rm user-authentication-app || true
                        docker stop mongo || true
                        docker rm mongo || true

                        # Run MongoDB container
                        docker run -d --name mongo --network auth-network -p 27017:27017 mongo:latest

                        # Run the app container with environment variables
                        docker run -d --name user-authentication-app --network auth-network -p 3000:3000 -e JWT_SECRET=\${JWT_SECRET} ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }
    post {
        always {
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
            sh "docker rmi ${IMAGE_NAME}:latest || true"
        }
    }
}