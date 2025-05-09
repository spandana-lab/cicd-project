Auth App
A simple Node.js application with signup and sign-in functionality, dockerized and integrated with AWS EC2, Jenkins, Docker, Trivy, SonarQube, and OWASP Dependency Check for a secure and automated CI/CD pipeline.
Table of Contents

Overview
Folder Structure
Prerequisites
Setup Instructions
Usage
CI/CD Pipeline
Security Tools
Environment Variables
Deployment
Contributing
License

Overview
This project is a Node.js (Express) application that provides user authentication (signup and sign-in) using MongoDB. It is containerized with Docker, deployed on an AWS EC2 instance, and integrated with Jenkins for CI/CD. Security is enhanced with Trivy for container image scanning, SonarQube for code quality analysis, and OWASP Dependency Check for dependency vulnerability scanning.
Folder Structure
auth-app/
├── .dockerignore       # Excludes files from Docker build context
├── .gitignore         # Excludes files from Git
├── Dockerfile         # Docker configuration for the Node.js app
├── Jenkinsfile        # Jenkins CI/CD pipeline script
├── docker-compose.yml # Orchestrates app and MongoDB containers
├── package.json       # Node.js dependencies and scripts
├── server.js          # Main application code
└── README.md          # Project documentation

Prerequisites

Local Development:
Node.js 18
Docker and Docker Compose
Git


Deployment:
AWS account with EC2 access
GitHub account
DockerHub account


CI/CD and Security:
Jenkins
Trivy
SonarQube
OWASP Dependency Check
Java 17 (for Jenkins, SonarQube, OWASP)



Setup Instructions

Clone the Repository:
git clone https://github.com/<YOUR_USERNAME>/auth-app.git
cd auth-app


Install Dependencies:
npm install


Run Locally with Docker:

Ensure Docker and Docker Compose are installed.
Start the app and MongoDB:docker-compose up --build


The app will be available at http://localhost:3000.


Set Up AWS EC2:

Launch an Ubuntu 22.04 LTS instance (t2.medium recommended).
Configure security group to allow:
SSH (port 22)
HTTP (port 80)
HTTPS (port 443)
App (port 3000)
Jenkins (port 8080)
SonarQube (port 9000)


Install dependencies on EC2:sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io openjdk-17-jdk
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu

Install Docker Compose, Jenkins, Trivy, SonarQube, and OWASP Dependency Check (refer to detailed steps in the project documentation).


Push to GitHub:
git add .
git commit -m "Initial commit"
git push origin main


Configure Jenkins:

Access Jenkins at http://<EC2_PUBLIC_IP>:8080.
Install plugins: Docker Pipeline, Git, NodeJS, SonarQube Scanner, OWASP Dependency-Check.
Configure NodeJS 18 and SonarQube server in Global Tool Configuration.
Add DockerHub credentials in Jenkins.
Create a pipeline job using the Jenkinsfile.



Usage

Signup:curl -X POST http://localhost:3000/signup -H "Content-Type: application/json" -d '{"email":"test@example.com","password":"password123"}'


Signin:curl -X POST http://localhost:3000/signin -H "Content-Type: application/json" -d '{"email":"test@example.com","password":"password123"}'


The sign-in endpoint returns a JWT token.

CI/CD Pipeline
The Jenkinsfile defines a pipeline with the following stages:

Checkout: Clones the GitHub repository.
Install Dependencies: Runs npm install.
SonarQube Analysis: Scans code quality.
OWASP Dependency Check: Scans for vulnerable dependencies.
Build Docker Image: Builds the Docker image.
Trivy Scan: Scans the image for vulnerabilities.
Push to DockerHub: Pushes the image to DockerHub.
Deploy: Deploys the app on EC2 using Docker Compose.

To trigger the pipeline:

In Jenkins, click "Build Now" on the auth-app-pipeline job.
Monitor logs and artifacts (e.g., OWASP reports).

Security Tools

Trivy: Scans Docker images for vulnerabilities (HIGH and CRITICAL severity).
SonarQube: Analyzes code quality and security issues at http://<EC2_PUBLIC_IP>:9000.
OWASP Dependency Check: Identifies vulnerable dependencies, with reports archived in Jenkins.

Environment Variables
Create a .env file (not tracked by Git) or set these in docker-compose.yml:

MONGO_URI: MongoDB connection string (e.g., mongodb://mongo:27017/authdb)
JWT_SECRET: Secret for JWT tokens (e.g., your_jwt_secret)

For production, use AWS Secrets Manager or similar.
Deployment

On EC2:
Copy the repository to the EC2 instance.
Run:docker-compose up -d


Access the app at http://<EC2_PUBLIC_IP>:3000.


Scaling: Consider AWS ECS or EKS for production-grade deployment.
Security:
Restrict Jenkins and SonarQube ports to your IP.
Enable HTTPS with AWS ALB or Nginx and Let’s Encrypt.



Contributing

Fork the repository.
Create a feature branch: git checkout -b feature-name.
Commit changes: git commit -m "Add feature".
Push to the branch: git push origin feature-name.
Open a pull request.

License
This project is licensed under the MIT License.
