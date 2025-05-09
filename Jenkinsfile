pipeline {
    agent any
    environment {
        // Define Docker Hub credentials ID (configured in Jenkins)
        DOCKER_CREDENTIALS_ID = 'spandu6677'
        // Docker image name
        IMAGE_NAME = 'spandu6677/cicd-project'
        // Tag with build number
        IMAGE_TAG = "${env.BUILD_NUMBER}"
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
                // Deploy the container (example: run on the same server)
                sh """
                    docker stop auth-app || true
                    docker rm auth-app || true
                    docker run -d --name auth-app -p 3000:3000 ${IMAGE_NAME}:${IMAGE_TAG}
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
/*pipeline {
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