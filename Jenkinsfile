pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "shubham4467"
            DOCKER_PASS = credentials('dockerhub')
            IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"	    
    }
    
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Shubham4676/register-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
       stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                sh "mvn sonar:sonar"
		        }
	          }
	     }	   
        }
	stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }       
        }
	stage("Build & Push Docker Image") {
            steps {
                script {
                    def dockerRegistryUrl = 'https://index.docker.io/v1/'
                    docker.withRegistry(dockerRegistryUrl, 'dockerhub') {
                        def dockerImage = docker.build("${IMAGE_NAME}")

                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }	    
}
