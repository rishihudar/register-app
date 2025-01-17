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
                    withSonarQubeEnv('Sonar') { // Ensure 'Sonar' is the correct server name in Jenkins configuration
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
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        
                        def dockerImageTag = "${IMAGE_NAME}:${RELEASE}-${BUILD_NUMBER}"
                        def latestTag = "${IMAGE_NAME}:latest"
                        
                        // Build Docker image
                        sh "docker build -t ${dockerImageTag} ."
                        
                        // Push the image with the version tag and 'latest'
                        sh "docker push ${dockerImageTag}"
                        sh "docker tag ${dockerImageTag} ${latestTag}"
                        sh "docker push ${latestTag}"
                    }
                }
            }
        }

        stage("Trivy: Filesystem Scan") {
            steps {
                script {
                    // Trivy scan example using shell commands
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${RELEASE}-${BUILD_NUMBER} --severity HIGH,CRITICAL
                    """
                }
            }
        }

        stage("Cleanup Artifacts") {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${RELEASE}-${BUILD_NUMBER} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh """
                        curl -v -k --user admin:${JENKINS_API_TOKEN} \
                        -X POST -H 'cache-control: no-cache' \
                        -H 'content-type: application/x-www-form-urlencoded' \
                        --data 'IMAGE_TAG=${RELEASE}-${BUILD_NUMBER}' \
                        'http://192.168.30.202:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'
                    """
                }
            }
        }
    }
}
