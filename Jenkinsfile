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
                    withSonarQubeEnv('Sonar') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("SonarQube: Code Quality Gates") {
            steps {
                script {
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
                    docker.withRegistry('', 'dockerhub-credentials-id') {
                        def docker_image = docker.build("${IMAGE_NAME}")
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trivy: Filesystem scan") {
            steps {
                script {
                    // Assuming trivy is installed and available on your Jenkins agent
                    sh "trivy image ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                script {
                    // Ensure the image is pushed successfully before removal
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
