pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    environment {
        AWS_REGION = 'eu-north-1'
        EKS_CLUSTER_NAME = 'my-cluster'
        AWS_ACCOUNT_ID = '145023095187'
    }
    stages {

        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/sagarkrishnasuresh/sample_project.git', branch: 'main'
            }
        }

        stage('Build JAR Files') {
            parallel {
                stage('Build User Management') {
                    steps {
                        dir('user_management') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
                stage('Build Order Management') {
                    steps {
                        dir('order_management') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
            }
        }

        stage('Run Ansible Playbook on EC2') {
            steps {
                script {
                    echo '🔹 Running Ansible Playbook to handle Docker build & push, and PostgreSQL setup...'
                    dir('Ansible') {
                        sh 'ansible-playbook -i inventory.ini complete_deployment.yml'
                    }
                }
            }
        }

        stage('Apply AWS ECR Secret in Kubernetes') {
            steps {
                script {
                    echo '🔹 Applying AWS ECR Kubernetes Secret...'
                    sh 'kubectl apply -f kubernetes/aws-ecr-secret.yml'
                }
            }
        }

        stage('Verify AWS EKS Cluster') {
            steps {
                script {
                    echo '🔹 Verifying EKS cluster is active...'
                    sh 'aws eks describe-cluster --name $EKS_CLUSTER_NAME --region $AWS_REGION --query "cluster.status"'
                }
            }
        }

        stage('Apply Kubernetes Secrets & Deploy Applications') {
            steps {
                script {
                    echo '🔹 Deploying user and order management apps to AWS EKS...'
                    sh '''
                    kubectl apply -f kubernetes/kubernetes-secrets.yml
                    kubectl apply -f kubernetes/user_management-deployment.yml
                    kubectl apply -f kubernetes/user_management-service.yml
                    kubectl apply -f kubernetes/order_management-deployment.yml
                    kubectl apply -f kubernetes/order_management-service.yml
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo '🔹 Checking if all pods and services are running...'
                    sh '''
                    kubectl get pods -o wide
                    kubectl get svc -o wide
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    echo '🧹 Cleaning up unnecessary files...'
                    sh '''
                    rm -rf user_management/target order_management/target
                    rm -f /var/lib/jenkins/tmp/kubernetes-secrets.yml
                    sudo docker system prune -a -f
                    '''
                    echo '✅ Cleanup completed successfully.'
                }
            }
        }
    }
    post {
        success {
            echo '✅ Build and deployment completed successfully on AWS EKS!'
        }
        failure {
            echo '❌ Build or deployment failed. Check logs for details.'
        }
    }
}
