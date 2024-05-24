pipeline {
    agent any
    tools {
        nodejs "node 22"
        dockerTool 'docker'
    }
    stages {
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/haythembekir20/todo-devops.git'
            }
        }
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('OWASP check-dependencies') {
            steps {
                sh "dependencyCheck additionalArguments: '-- scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dp-check'"
                sh "dependencyCheckPublisher pattern: '**/dependency-check-report.xml'"
            }
        }
        stage("Build") {
            steps {
                script {
                    docker.build("haythembekir20/todo-devops:${env.BUILD_NUMBER}")
                }
            }
        }
        stage("Push") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("haythembekir20/todo-devops:${env.BUILD_NUMBER}").push()
                        docker.image("haythembekir20/todo-devops:${env.BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage("Deploy") {
            steps {
                sh "kubectl apply -f k8s/"
                sh "kubectl rollout status deployment/todo-devops"
                sh "kubectl get pods"
            }
        }
    }
}