pipeline {
    agent { label 'vinod' }
    tools {
        jdk 'Java17' // Ensure Java 17 is configured in Global Tool Configuration
        maven 'Maven3' // Ensure Maven 3 is configured in Global Tool Configuration
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
                    sonarqube_code_quality()
                }
            }
        } 
    }
}
