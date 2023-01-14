pipeline{
    agent any 
    environment {
        VERSION = "${env.BUILD_ID}"
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
                     withSonarQubeEnv(credentialsId: 'Sonar_token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }                  
                    timeout(time: 1, unit: 'HOURS'){
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                
                }
            }
        }
        stage ("Docker Build and Docker push"){
            steps{
                script{ 
                  //withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                    sh '''
                      docker build -t 34.125.181.2:8083/springapp:${VERSION} .
                      docker login -u admin -p password 34.125.181.2:8083
                      docker push  34.125.181.2:8083/springapp:${VERSION}
                      docker rmi  34.125.181.2:8083/springapp:${VERSION}
                    ''' 
                  
                } 
            } 
        }

        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/'){
                        sh 'helm datree test myapp/ --no-record'
                    }
                }
            }
        }
     
    }
  
 }
