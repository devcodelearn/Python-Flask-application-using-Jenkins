pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials' // Define your DockerHub credentials in Jenkins
        DOCKER_IMAGE_NAME = 'your-dockerhub-username/flask-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Cloning the repository
                git 'https://github.com/your-github-username/flask-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install dependencies
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                // Run tests using pytest
                sh 'pytest --maxfail=1 --disable-warnings'
            }
        }

        stage('Dockerize the App') {
            steps {
                // Build the Docker image
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                // Login to DockerHub and push the Docker image
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                // Remove any existing container and run a new one
                sh """
                docker stop flask-app || true && docker rm flask-app || true
                docker run -d -p 5000:5000 --name flask-app ${DOCKER_IMAGE_NAME}:latest
                """
            }
        }
    }

    post {
        always {
            // Clean up the workspace
            cleanWs()
        }
    }
}
