pipeline {
    agent any
    stages {
        stage('clone repo'){
            steps {
                git 'https://github.com/zeru427/sample-spring-boot.git'
                script {
                    git branch: 'pipelinetesting', url: "https://github.com/zeru427/sample-spring-boot.git" 
                }
            }
        }

        stage('Maven Build - Package') {
            steps {
                script {
                    def mvnHome = tool 'MyMaven'
                    mvnHome = tool 'MyMaven'
                    echo "Packaging application"
                    sh "${mvnHome}/bin/mvn clean install"
                }
            }
        }
        
        stage('Docker Image Build') {
            steps {
                script {
                    echo "Building docker image..."
                    dockerImage = docker.build("zeruyang001/zerudockerhub:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Build docker deploy'){
            steps {
                script {
                    echo "Uploading Docker image to Dockerhub"
                    //docker-hub-credentials - we have to create in jenkins credentials
                    docker.withRegistry('https://registry.hub.docker.com/','docker-hub-credentials') {
                        dockerImage.push("${env.BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Deploying App to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(configs: "kubernetes.yml", kubeconfigId: "kubernetes")
                }
            }
        }
        stage('deploying production to kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'sre-lab', contextName: 'arn:aws:eks:us-east-1:855430746673:cluster/sre-lab', credentialsId: 'k8s', namespace: 'zeru-yang', serverUrl: 'https://8175C01E797F39C77ED8AB94CD24986B.gr7.us-east-1.eks.amazonaws.com') {
                    sh ('kubectl apply -f kubernetes.yml')
                    sh ('kubectl get all')
                }
            }
        }
        

    }

}
