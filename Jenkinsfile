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
                    args '--network host' // Allows access to SonarQube server
                }
            }
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarScanner') {
                        sh """
                            sonar-scanner \
                                -Dsonar.projectKey=cicd-project \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://54.203.99.36:9000 \
                                -Dsonar.login=\${SONAR_TOKEN}
                        """
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
        stage('Trivy Scan') {
            steps {
                sh """
                    trivy image --exit-code 1 --severity HIGH,CRITICAL --no-progress ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME}:${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
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
    post {
        always {
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
    }
}

/*pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'f84d68c0-d4b6-461e-8890-bd5e45e5cb37'  // Matches your Jenkins credentials ID
        IMAGE_NAME = 'spandu6677/cicd-project'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        //SONARQUBE_SERVER = 'mysonar'
        //DOCKER_BUILDKIT = '1'  // Enable BuildKit
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
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('mysonar') {
                        sh """
                        sonar-scanner \
                        -Dsonar.projectKey=cicd-project \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://54.203.99.36:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
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
        stage('Trivy Scan') {
            steps {
                sh """
                    trivy image --exit-code 1 --severity HIGH,CRITICAL --no-progress ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                //withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
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
                //}
            }
        }
    }
    post {
        always {
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
    }
}*/







/*pipeline {
    agent any
    environment {
        // Define Docker Hub credentials ID (configured in Jenkins)
        DOCKER_CREDENTIALS_ID = 'f84d68c0-d4b6-461e-8890-bd5e45e5cb37'
        // Docker image name
        IMAGE_NAME = 'spandu6677/cicd-project'
        // Tag with build number
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        //DOCKER_BUILDKIT = '1'  // Enable BuildKit
    }
    stages {
        stage('Checkout') {
            steps {
                // Pull code from Git
                git url: 'https://github.com/spandana-lab/cicd-project.git', branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                // Build Docker image
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                // Login and push image to Docker Hub
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        // Optionally push 'latest' tag
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
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
                docker stop auth-app-mango || true
                docker rm auth-app-mango || true
                docker stop auth-app-mango || true
                docker rm auth-app-mango || true
                
                # Run MongoDB container
                docker run -d --name auth-app-mango --network auth-network -p 27017:27017 mongo:latest
                
                # Run the app container, linking to the MongoDB network
                docker run -d --name auth-app --network auth-network -p 3000:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            // Clean up Docker images locally
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
    }
}





pipeline {
    agent any
    tools {
        nodejs 'node24'
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('spandu6677')
        SONARQUBE_SERVER = 'SonarQube'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/spandana-lab/cicd-project.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner -Dsonar.projectKey=auth-app -Dsonar.sources=. -Dsonar.host.url=http://54.203.99.36:9000 -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                sh '/opt/dependency-check/bin/dependency-check.sh --project auth-app --scan . --out dependency-check-report.html --format HTML'
                dependencyCheckPublisher pattern: 'dependency-check-report.html'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t spandu6677/auth-app:latest .'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL spandu6677/auth-app:latest'
            }
        }
        stage('Push to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push spandu6677/auth-app:latest'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker-compose down'
                sh 'docker-compose up -d'
            }
        }
    }
    post {
        always {
            node '' {
                archiveArtifacts artifacts: 'dependency-check-report.html', allowEmptyArchive: true
            }
        }
    }
}
*/