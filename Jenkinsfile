def ProjectId="prrana"
pipeline{
    environment {
    registry = "prrana/internal"
    registryCredential = 'dockerhub'
    dockerImage = ''
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
                     dockerImage = docker.build registry + ":V5.0.$BUILD_NUMBER"
        
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
                    gcloud container clusters get-credentials demo-app-cluster --zone us-central1-a --project prrana
                    kubectl set image deployment/events-data events-data=$registry:V5.0.$BUILD_NUMBER --namespace=internal-external
                """
            }    
        } 
        stage('Remove Unused docker image') {
             steps{
              sh "docker rmi $registry:V5.0.$BUILD_NUMBER"
                }
            }
            
}			
}


