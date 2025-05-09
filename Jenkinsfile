pipeline {
    agent any
    tools {
        nodejs 'Node18'
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        SONARQUBE_SERVER = 'SonarQube'
    }
    stages {
        stage('Checkout') {
            steps {
<<<<<<< HEAD
                git '<GITHUB_REPO_URL>'
=======
                branch '*/main'
                git 'https://github.com/spandana-lab/cicd-project.git'
>>>>>>> feee84df51d0cb54ee5d71ee9f61d8691e89c09e
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
                    sh 'sonar-scanner -Dsonar.projectKey=auth-app -Dsonar.sources=. -Dsonar.host.url=http://<EC2_PUBLIC_IP>:9000 -Dsonar.login=$SONAR_TOKEN'
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
                sh 'docker build -t <DOCKERHUB_USERNAME>/auth-app:latest .'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL <DOCKERHUB_USERNAME>/auth-app:latest'
            }
        }
        stage('Push to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push <DOCKERHUB_USERNAME>/auth-app:latest'
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
            archiveArtifacts artifacts: 'dependency-check-report.html', allowEmptyArchive: true
        }
    }
}
