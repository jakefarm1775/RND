// Define the 'app' variable at the top level to be accessible across stages
// and to prevent the 'def' keyword warning.
def app

pipeline {
    agent any

    environment {
        // Docker Hub credentials ID stored in Jenkins
        DOCKERHUB_CREDENTIALS = 'RND'
        IMAGE_NAME = 'jakefarm1775/rnd:latest'
        // Create a unique project name for each build to avoid conflicts
        COMPOSE_PROJECT_NAME = "rnd-${env.BUILD_NUMBER}"
    }

    stages {

        stage('Cloning Git') {
            steps {
                checkout scm
            }
        }

        stage('SAST') {
            steps {
                sh 'echo Running SAST scan...'
            }
        }

        stage('BUILD-AND-TAG') {
            agent {
                label 'appserver'
            }
            steps {
                script {
                    // Build Docker image using Jenkins Docker Pipeline API
                    echo "Building Docker image ${IMAGE_NAME}..."
                    // Assign the built image to the pre-defined 'app' variable
                    app = docker.build("${IMAGE_NAME}")
                    app.tag("latest")
                }
            }
        }


        stage('POST-TO-DOCKERHUB') {    
            agent {
                label 'appserver'
            }
            steps {
                script {
                    echo "Pushing image ${IMAGE_NAME}:latest to Docker Hub..."
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKERHUB_CREDENTIALS}") {
                        app.push("latest")
                    }
                }
            }
        }

        stage('SECURITY-IMAGE-SCANNER') {
            steps {
                sh 'echo Scanning Docker image for vulnerabilities...'
            }
        }

        stage('Pull-image-server') {
            steps {
                sh 'echo Pulling image on server...'
            }
        }

        stage('DAST') {
            steps {
                sh 'echo Performing DAST scan...'
            }
        }

        stage('DEPLOYMENT') {    
            agent {
                label 'appserver'
            }
            // Use a post step with 'always' to ensure cleanup happens even if the build fails
            post {
                always {
                    script {
                        dir("${WORKSPACE}") {
                            echo "Cleaning up Docker Compose environment: ${COMPOSE_PROJECT_NAME}"
                            // Use the unique project name to tear down the specific environment
                            sh "docker compose --project-name ${COMPOSE_PROJECT_NAME} down --remove-orphans"
                        }
                    }
                }
            }
            steps {
                echo "Starting deployment using docker-compose project: ${COMPOSE_PROJECT_NAME}"
                script {
                    dir("${WORKSPACE}") {
                        sh """
                            # Use the unique project name to bring up the environment
                            docker compose --project-name ${COMPOSE_PROJECT_NAME} up -d
                            
                            # Give containers a moment to start
                            sleep 5 
                            
                            echo "Current running containers for this project:"
                            docker compose --project-name ${COMPOSE_PROJECT_NAME} ps
                        """
                    }
                }
                echo 'Deployment completed successfully!'
            }
        }
    }   
}

