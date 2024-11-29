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
                kubernetesApply(
                    CredentialsId: 'kubeconfig',
                    file: 'train-schedule-kube-canary.yml',
                
                )
            }
        }
        stage('DeployToProduction') {
           
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesApply(
                    CredentialsId: 'kubeconfig',
                    file: 'train-schedule-kube-canary.yml',
                
                )
                kubernetesApply(
                   CredentialsId: 'kubeconfig',
                     file: 'train-schedule-kube.yml'
                    
                )
            }
        }
    }
}
