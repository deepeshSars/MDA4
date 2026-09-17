pipeline {
    agent any
 
    environment {
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
        REGISTRY = 'docker.io/dipu12'
        aws_access_key = credentials('aws-access-key')
        aws_secret_key = credentials('aws-secret-key')
    }

    options {
        skipDefaultCheckout false
    }

    stages {
        stage("Pull stage") {
            steps {
                sh 'rm -rf MDA4 || true'
                sh 'git clone https://github.com/deepeshSars/MDA4.git'
             }
          }
        }

        stage("Infrastructure") {
            steps {
                dir('Terraform/eks-modules') {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage("Build") {
            steps {
                dir('docker/student-app/database') {
                    sh 'docker build -t studentapp-db .'
                }
                dir('docker/student-app/backend') {
                    sh 'docker build -t studentapp-be .'
                }
                dir('docker/student-app/frontend') {
                    sh 'docker build -t studentapp-fe .'
                }
            }
        }

        stage("Push") {
            steps {
                sh 'docker push studentapp-db'
                sh 'docker push studentapp-be'
                sh 'docker push studentapp-fe'
            }
        }

        stage("Deploy") {
            steps {
                dir('Kubernetes/student-app') {
                    sh 'kubectl apply -f .'
                }
            }
        }
    }
}
