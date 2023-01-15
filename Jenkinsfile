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
                     withSonarQubeEnv(credentialsId: 'Sonartoken') {
                         // Sonartoken: this is sonar pass that we saved in Jenkins
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
                  withCredentials([string(credentialsId: 'DockerPass', variable: 'docker_password')]) {
                    sh '''
                      docker build -t 34.135.143.124:8083/springapp:${VERSION} .
                      docker login -u admin -p $docker_password 34.135.143.124:8083
                      docker push  34.135.143.124:8083/springapp:${VERSION}
                      docker rmi  34.135.143.124:8083/springapp:${VERSION}
                    ''' 
                  }
                } 
            } 
        }

        stage ('Identifying misconfigutaion using datree in helm'){
             steps{
                 script{
                      dir('kubernetes/'){
                          withEnv(['DATREE_TOKEN=6f8b1d0c-26eb-4d75-88cd-cc349c86210a']) {
                              sh 'helm datree test myapp/ --no-record'
                          }
                          
                      }
                 }
                 }
             }
        stage ("Pushing the helm chart to nexus"){
            steps{
                script{ 
                  withCredentials([string(credentialsId: 'DockerPass', variable: 'docker_password')]) {
                   dir('kubernetes/'){    
                    sh '''
                      helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                      tar -czvf myapp-${helmversion}.tgz myapp/
                      curl -u admin:$docker_password http://34.135.143.124:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v    
                    ''' 
                   }
                  }
                } 
            } 
        }     
     
    }
  
 }
