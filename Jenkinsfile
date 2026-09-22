pipeline {
    agent any

    environment {
        REGISTRY = 'docker.io/dipu12'
        aws_access_key = credentials('aws-access-key')
        aws_secret_key = credentials('aws-secret-key')
    }

    options {
        skipDefaultCheckout false
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/deepeshSars/MDA4.git'
            }
        }

        stage("Infrastructure") {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve -var="aws_access_key=${aws_access_key}" -var="aws_secret_key=${aws_secret_key}" -var-file="terraform.tfvars"'
                }
            }
        }

        stage("Build") {
            steps {
                dir('docker/database') {
                    sh 'docker build --build-arg BUILDKIT_INLINE_CACHE=1 -t ${REGISTRY}/studentapp-db .'
                }
                dir('docker/backend') {
                    sh 'docker build --build-arg BUILDKIT_INLINE_CACHE=1 -t ${REGISTRY}/studentapp-be .'
                }
                dir('docker/frontend') {
                    sh 'docker build --build-arg BUILDKIT_INLINE_CACHE=1 -t ${REGISTRY}/studentapp-fe .'
                }
            }
        }

        stage("Push") {
            steps {
                sh 'docker push ${REGISTRY}/studentapp-db'
                sh 'docker push ${REGISTRY}/studentapp-be'
                sh 'docker push ${REGISTRY}/studentapp-fe'
            }
        }

        stage("Deploy") {
            steps {
                dir('kubernetes/Database') {
                    sh 'kubectl apply -f .'
                }
                dir('kubernetes/Backend') {
                    sh 'kubectl apply -f .'
                }
                dir('kubernetes/Frontend') {
                    sh 'kubectl apply -f .'
                }
            }
        }
    }
}