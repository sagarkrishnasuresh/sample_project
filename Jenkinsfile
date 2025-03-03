pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    environment {
        AWS_REGION = 'eu-north-1'
        EKS_CLUSTER_NAME = 'my-cluster'
        AWS_ACCOUNT_ID = '145023095187'
        REPO_NAME_USER = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/user_management"
        REPO_NAME_ORDER = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/order_management"
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

        stage('Login to AWS ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                        sh '''
                        echo "Logging into AWS ECR..."
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                        '''
                    }
                }
            }
        }


        stage('Build & Push Docker Images') {
            parallel {
                stage('Build & Push User Management') {
                    steps {
                        sh '''
                        docker build -t $REPO_NAME_USER:latest ./user_management
                        docker push $REPO_NAME_USER:latest
                        '''
                    }
                }
                stage('Build & Push Order Management') {
                    steps {
                        sh '''
                        docker build -t $REPO_NAME_ORDER:latest ./order_management
                        docker push $REPO_NAME_ORDER:latest
                        '''
                    }
                }
            }
        }

        stage('Run Ansible Playbook on EC2') {
            steps {
                dir('Ansible') {
                    sh 'ansible-playbook -i inventory.ini complete_deployment.yml'
                }
            }
        }

        stage('Apply Kubernetes Secrets & Deploy') {
            steps {
                script {
                    echo 'Applying Kubernetes secrets and manifests...'
                    sh '''
                    kubectl apply -f kubernetes/aws-ecr-secret.yml
                    kubectl apply -f kubernetes/kubernetes-secrets.yml
                    kubectl apply -f kubernetes/postgres-deployment.yml
                    kubectl apply -f kubernetes/postgres-service.yml
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
                    echo 'Checking if all pods and services are running...'
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
                    echo 'Cleaning up unnecessary files...'
                    sh '''
                    rm -rf user_management/target order_management/target
                    rm -f Ansible/roles/postgres_setup/templates/kubernetes-secrets.yml
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
