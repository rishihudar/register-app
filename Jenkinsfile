pipeline {
    agent { label 'vinod' }
    tools {
        jdk 'Java17' // Ensure Java 17 is configured in Global Tool Configuration
        maven 'Maven3' // Ensure Maven 3 is configured in Global Tool Configuration
    }
   environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "rishi9759"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
     }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/rishihudar/register-app'
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

        stage("SonarQube: Code Analysis") {
            steps {
                script {
                    // This should use the SonarQube plugin and environment configured in Jenkins
                    withSonarQubeEnv('Sonar') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("SonarQube: Code Quality Gates") {
            steps {
                script {
                    // Wait for SonarQube quality gate status
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "SonarQube quality gate failed: ${qualityGate.status}"
                    }
                }
            }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        
        }
    }
}
