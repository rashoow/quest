pipeline {
    
    agent any
    
    tools {
        maven 'Maven3'
    }

   environment {
        //once you sign up for Docker hub, use that user_id here
        registry = "rashoow/rearc"
        //- update your credentials ID after creating credentials for connecting to Docker Hub
        registryCredential = 'e5b70cd2-4c19-4415-973d-3ac99b087041'
        dockerImage = ''
        // Docker TAG
        DOCKER_TAG = getVersion()
    }

    stages {
        stage('Checkout') {
            steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'd1bd1c65-9332-42fa-9e4a-845fa17bf17f', url: 'https://github.com/rashoow/quest']]])
            }
        }
        
    stage ('Build docker image') {
            steps {
                sh "docker build . -t $registry:${DOCKER_TAG}"
                
            }
       }
    
    // Uploading Docker images into Docker Hub
    stage('Upload Image') {
            steps{   
                script {
                    docker.withRegistry( '', registryCredential ) {
                        sh "docker push $registry:${DOCKER_TAG}"
            }
       }
    }
}

    stage('Cleaning up') {
            steps{
                sh "docker rmi $registry:${DOCKER_TAG}"
            }
        }
        
    stage('Docker Run') {
            steps{
                script {
                    sh 'docker run --name rearc-app -d -p 8096:3000 --rm $registry:${DOCKER_TAG}'
                }
            }
        }
        
       stage ('K8S Deploy') {
            steps {
                script {
            withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', serverUrl: '']]) {
                sh "kubectl apply -f kubernetes_files"
                    }
                }
            }
        }        
    }
}    

def getVersion() {
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
    



