pipeline {
    agent any
    environment{
        registry= "toncheto/javaapp"
        registryCredential = 'dockerhub_id'
        
    }
    
    stages{
        stage('Build Maven'){            
            agent {                
                docker {
                    image 'maven:3.8.1-adoptopenjdk-11'
                    args '-v /root/.m2:/root/.m2'
                }
             }
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/AntonZl/devops-automation']]])
                sh 'mvn -B -DskipTests clean package'
            }
            post {
              success {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
              }
            }
        }
        stage('Test'){            
            agent {                
                docker {
                    image 'maven:3.8.1-adoptopenjdk-11'
                    args '-v /root/.m2:/root/.m2'
                }
             }
            steps{                
                sh 'mvn test'
            }
        }        
        
        stage('Build docker image'){
            agent {
                docker { image 'docker:20.10.21-alpine3.17' }            
            }
            steps{
                script{
                    sh '''
                    docker build -t "$registry:$BUILD_NUMBER" .
                    echo "Listing all images:"
                    docker images
                    '''
                }
            }
        }
        stage('Push image to Dockerhub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                   sh '''
                       docker login -u toncheto -p "$dockerhubpwd"
                   '''

                }
                   sh '''
                   docker push "$registry:$BUILD_NUMBER"
                   '''
                }
            }
        }
        
        stage('Deploy to Windows Docker'){
            steps{
                script{
                    sh '''
                    docker run -d --rm -p 8087:8080 --name javaapp "$registry:$BUILD_NUMBER"
                    '''
                }
            }
        }
    }
}
