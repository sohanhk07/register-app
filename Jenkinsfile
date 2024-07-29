pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME    = "register-app-pipeline"
        RELEASE     = "1.0.0"
        DOCKER_USER = "shubham4467"
        // Use credentials ID for DOCKER_PASS
        DOCKER_PASS = credentials('dockerhub')
        IMAGE_NAME  = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG   = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Shubham4676/register-app'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    def qualityGate = waitForQualityGate(
                        abortPipeline: false,
                        credentialsId: 'jenkins-sonarqube-token',
                        timeout: 10 * 60  // Timeout in seconds (10 minutes)
                    )
                    if (qualityGate.status != 'OK') {
                        error "SonarQube Quality Gate failed: ${qualityGate.status}"
                    }
                }
            }
        }


        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Define Docker registry URL for Docker Hub
                    def dockerRegistryUrl = 'https://index.docker.io/v1/'

                    // Build the Docker image
                    docker.withRegistry(dockerRegistryUrl, 'dockerhub') {
                        def dockerImage = docker.build("${IMAGE_NAME}")

                        // Push the Docker image with specific tag and latest tag
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }
}
