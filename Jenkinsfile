def ProjectId="prrana"
pipeline{
    environment {
    registry = "prrana/internal"
    registryCredential = 'dockerhub'
    dockerImage = ''
    Image_name = "${ProjectId}/internal:v1.0.${BUILD_ID}"
  }
    agent any 
    stages{
        stage('dependency versions'){
            steps{
                sh '''
                    git --version
                    docker --version
                    npm -v
                '''
            }    
        }
        stage('git checkout'){
            steps{
                    git 'https://github.com/prrana19/internal-repo.git'
            }    
        }
        stage('git test'){
            steps{
                sh '''
                    ls -a
                    echo "install dependencies and test internal code ..!"
                    npm install
                    npm test
                ''' 
            }    
        }
        stage('Sonarqube') {
                environment {
                    scannerHome = tool 'SonarScanner'
                     }
                steps {
                     withSonarQubeEnv('sonarQube') {
                     sh "${scannerHome}/bin/sonar-scanner"
                      }
                 }
        }
        
        stage('building image'){
            steps{
                 script {
                     dockerImage = docker.build registry + "$BUILD_NUMBER"
        
                        }
                     }
             }
        stage('Push Image to dockerhub') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push()
                            }
                         }
                    }
            }
            stage('deploy'){
            steps{
                sh """
                    gcloud container clusters get-credentials my-app-cluster --zone us-central1-a --project prrana
                    kubectl set image deployment/events-data events-data=$registry$BUILD_NUMBER --namespace=internal-external
                """
            }    
        } 
        stage('Remove Unused docker image') {
             steps{
              sh "docker rmi $registry$BUILD_NUMBER"
                }
            }
            
}			
}


