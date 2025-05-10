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
                // Ensure full clone with .git directory
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/spandana-lab/cicd-project.git']],
                    extensions: [[$class: 'CloneOption', noTags: false, shallow: false, depth: 0]]
                ])
                // Verify .git directory after checkout
                sh 'ls -la .git || echo "No .git directory found in primary workspace"'
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
                        script {
                            try {
                                sh '''
                                    # Verify Git repository
                                    ls -la ${WORKSPACE}/.git || echo "No .git directory found in Docker workspace"
                                    git status || echo "Git status failed"
                                    export SONAR_USER_HOME=${WORKSPACE}/.sonar
                                    sonar-scanner \
                                        -Dsonar.projectKey=cicd-project \
                                        -Dsonar.sources=. \
                                        -Dsonar.host.url=http://54.203.99.36:9000 \
                                        -Dsonar.scm.disabled=true \
                                        -Dsonar.scanner.socketTimeout=120 \
                                        -Dsonar.inclusions=**/*.js,**/*.ts
                                    ls -l ${SONAR_USER_HOME} || echo "Sonar cache directory not found"
                                    ls -l .scannerwork || echo "Scanner work directory not found"
                                '''
                            } catch (Exception e) {
                                echo "SonarQube scan failed: ${e.message}"
                                error "Stopping pipeline due to SonarQube failure"
                            }
                        }
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
                sh """
                    # Create a Docker network for the app and MongoDB
                    docker network create auth-network || true

                    # Stop and remove existing containers
                    docker stop user-authentication-app || true
                    docker rm user-authentication-app || true
                    docker stop mongo || true
                    docker rm mongo || true
                    docker stop sonarqube || true
                    docker rm sonarqube || true

                    # Run MongoDB container
                    docker run -d --name mongo --network auth-network -p 27017:27017 mongo:latest

                    # Run the app container with environment variables
                    docker run -d --name user-authentication-app --network auth-network -p 3000:3000 -e JWT_SECRET='GciOiJIdfhey763fdiwe87d6gdisj1NiIc6IkpXV' ${IMAGE_NAME}:${IMAGE_TAG}
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