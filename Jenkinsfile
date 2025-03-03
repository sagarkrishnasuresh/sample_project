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
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-credentials',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh '''
                        echo "✅ Logging into AWS ECR..."
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                        '''
                    }
                }
            }
        }

        stage('Ensure ECR Repositories Exist') {
            steps {
                script {
                    echo '🔹 Checking if AWS ECR repositories exist...'
                    sh '''
                    set -e
                    if ! aws ecr describe-repositories --repository-names user_management --region $AWS_REGION >/dev/null 2>&1; then
                        aws ecr create-repository --repository-name user_management --region $AWS_REGION
                    fi
                    if ! aws ecr describe-repositories --repository-names order_management --region $AWS_REGION >/dev/null 2>&1; then
                        aws ecr create-repository --repository-name order_management --region $AWS_REGION
                    fi
                    '''
                }
            }
        }

        stage('Build & Push Docker Images') {
            parallel {
                stage('Build & Push User Management') {
                    steps {
                        sh '''
                        docker build --no-cache -t $REPO_NAME_USER:latest ./user_management
                        docker push $REPO_NAME_USER:latest
                        '''
                    }
                }
                stage('Build & Push Order Management') {
                    steps {
                        sh '''
                        docker build --no-cache -t $REPO_NAME_ORDER:latest ./order_management
                        docker push $REPO_NAME_ORDER:latest
                        '''
                    }
                }
            }
        }

        stage('Apply AWS ECR Secret in Kubernetes') {
            steps {
                script {
                    echo '🔹 Applying AWS ECR Kubernetes Secret...'
                    sh '''
                    kubectl apply -f kubernetes/aws-ecr-secret.yml
                    '''
                }
            }
        }

        stage('Verify AWS EKS Cluster') {
            steps {
                script {
                    echo '🔹 Verifying EKS cluster is active...'
                    sh '''
                    aws eks describe-cluster --name $EKS_CLUSTER_NAME --region $AWS_REGION --query "cluster.status"
                    '''
                }
            }
        }

        stage('Apply Kubernetes Secrets & Deploy') {
            steps {
                script {
                    echo '🔹 Deploying application manifests to AWS EKS...'
                    sh '''
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
