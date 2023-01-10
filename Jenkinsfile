pipeline{
    agent any 
    environment {
        VERISON = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'Sonar_Qube') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                   /* 
                    timeout(time: 1, unit: 'HOURS'){
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }*/
                }
            }
        }
        stage ("Docker Build and Docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                    sh '''
                      docker build --tag 34.125.18.109:8083/springapp:${VERSION} .
                      docker login -u admin -p $docker_password 34.125.18.109:8083
                      docker push  34.125.18.109:8083/springapp:${VERSION}
                      docker rmi  34.125.18.109:8083/springapp:${VERSION}
                    '''
    
                }
                   
                }
            }
        }
    }
 }
