pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "sharda11/train-schedule"
    }
    stages {
       
        stage('Build Docker Image') {
            
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        
        stage('CanaryDeploy') {
            
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                
                milestone(1)
                withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://35.200.162.22']) {
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                    sh 'kubectl apply -f train-schedule-kube.yml'
                  
 
                   
                }
            }
         }
        stage('DeployToProduction') {
           
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                
                milestone(1)
                 withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://35.200.162.22']) {
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                    sh 'kubectl apply -f train-schedule-kube.yml'
                  

                }
        }
    }
}
}
