pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')  // optional for other API use
        DOCKER_REGISTRY = 'pavankumar0185' // Replace with your Docker Hub username or registry URL
        IMAGE_NAME = 'java-car-game'                // Docker image name
        TAG = "${BUILD_NUMBER}-${GIT_COMMIT}"       // Tag Docker image with build number and commit hash
        DOCKER_CREDENTIALS = 'dockerhub-creds'   // Jenkins credentials ID for Docker registry
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/BhojanapuPavanKumar/java-app.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'pwd && ls -la'               // List directory contents to verify repo
                sh 'mvn clean install'           // Build the Java project using Maven
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${TAG}"
                    // Build Docker image with a tag using the build number and Git commit
                    sh "docker build -t ${imageTag} ."
                }
            }
        }

        stage('Publish Docker Image') {
            steps {
                script {
                    // Log into Docker registry and push the image
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS}") {
                        sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${TAG}"
                    }
                }
            }
        }

        stage('Deploy App') {
            steps {
                script {
                    // Stop old container if running (optional cleanup)
                    sh 'docker stop java-car-game-container || true'
                    sh 'docker rm java-car-game-container || true'

                    // Run new container on port 8081 (since 8080 is used by Jenkins)
                    sh 'docker run -d --name java-car-game-container -p 8081:8080 java-car-game'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Verify the deployment by checking HTTP response
                    def response = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://localhost:8081', returnStdout: true).trim()
                    if (response != '200') {
                        error "Deployment failed with status code: ${response}"
                    } else {
                        echo "Deployment verified successfully!"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
            cleanWs()  // Clean up workspace after the build
        }
    }
}
