pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'devcodelearn/Python-Flask-application-using-Jenkins'
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Clone the repository from GitHub using credentials
                withCredentials([string(credentialsId: 'Jenkins_PAT', variable: 'GITHUB_PAT')]) {
                    git branch: 'main',
                        url: "https://${GITHUB_PAT}@github.com/devcodelearn/Python-Flask-application-using-Jenkins.git"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Python dependencies
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                // Run unit tests using pytest
                sh 'pytest --maxfail=1 --disable-warnings'
            }
        }

        stage('Dockerize the App') {
            steps {
                script {
                    // Build Docker image
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    // Use Jenkins credentials to log in to DockerHub
                    withCredentials([usernamePassword(credentialsId: 'Docker_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_TOKEN')]) {
                        // Log in to DockerHub using username and secret token (key)
                        sh "echo $DOCKER_TOKEN | docker login -u $DOCKER_USER --password-stdin"
                        
                        // Push the Docker image to DockerHub
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    // Deploy the application as a Docker container
                    sh """
                    docker stop Python-Flask-application-using-Jenkins || true && docker rm Python-Flask-application-using-Jenkins || true
                    docker run -d -p 5000:5000 --name Python-Flask-application-using-Jenkins ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
