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
        DOCKER_PASS = 'dockerhub' // Sensitive information should be stored securely (e.g., Jenkins credentials store)
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Log in to Docker registry using credentials
                    docker.withRegistry('', "${DOCKER_PASS}") {
                        // Build Docker image
                        def docker_image = docker.build("${IMAGE_NAME}")
                        
                        // Push the image with the version tag and 'latest'
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    // Run Trivy scan to check for vulnerabilities
                    sh 'docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image rishi9759/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table'
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                script {
                    // Remove Docker images after scan and push
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' '192.168.30.202:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}
