pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"     
    }
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
withSonarQubeEnv(credentialsId: 'jenkins_sonar') {
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
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                      sh'''
                    docker build -t 35.232.95.226:8083/springapp:${VERSION} .
                    docker login -u admin -p $docker_password 35.232.95.226:8083
                    docker push 35.232.95.226:8083/springapp:${VERSION}
                    docker rmi 35.232.95.226:8083/springapp:${VERSION}
                    '''
                    }
                }
            }
        }
    }
}