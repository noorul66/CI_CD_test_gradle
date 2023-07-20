pipeline {
    agent any

    stages {
        stage("Sonar Quality Check") {
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage("Docker Build & Push") {
            agent any
            steps {
                script {
                    // Your Docker build and push steps here
                    // Make sure to replace the placeholders with your actual commands
                }
            }
        }

        // Add more stages as needed for your pipeline
    }

    post {
        always {
            // Your post-build actions here
            // For example, sending notifications or cleaning up resources
        }
    }
}
