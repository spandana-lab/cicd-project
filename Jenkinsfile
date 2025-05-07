pipeline {
    agent any

    tools {
        nodejs "node24"
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
                branch '*/main'
                git 'https://github.com/spandana-lab/cicd-project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t devops-app .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image devops-app || true' // Ignore non-zero exit if vulnerabilities found
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 3000:3000 --name devops-container devops-app'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
