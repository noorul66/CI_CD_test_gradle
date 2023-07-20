pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
    }

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

        stage('pushing the helm charts to nexus') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        dir('kubernetes/') {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                            tar -czvf myapp-\${helmversion}.tgz myapp/
                            curl -u admin:\$docker_password http://34.171.89.251:8081/repository/helm-hosted/ --upload-file myapp-\${helmversion}.tgz -v
                            '''
                        }
                    }
                }
            }
        }

        stage("docker build & docker push") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                            docker build -t 34.171.89.251:8083/springapp:${VERSION} .
                            docker login -u admin -p $docker_password 34.171.89.251:8083 
                            docker push 34.171.89.251:8083/springapp:${VERSION}
                            docker rmi 34.171.89.251:8083/springapp:${VERSION}
                        '''
                    }
                }
            }
        }

        // Add more stages as needed for your pipeline
    }

    post {
        always {
            mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";  
        }
    }
}
