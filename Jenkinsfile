pipeline {
    agent any
    stages {
        stage('Get source') {
            steps {
                git url: 'https://github.com/andre-mf/deploy-jenkins-k8s-registry', branch: 'main'
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    dockerapp = docker.build("localhost:5000/hello-world-nginx:${env.BUILD_ID}",
                    '-f Dockerfile .')
                }
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    docker.withRegistry('https://localhost:5000', 'Registry') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('Deploy Kubernetes') {
            agent {
                kubernetes {
                    cloud 'kubernetes'
                }
            }
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
            }
        }
    }
}
