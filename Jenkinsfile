pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    environment {
        AWS_REGION = 'eu-north-1'
        EKS_CLUSTER_NAME = 'my-cluster'
        AWS_ACCOUNT_ID = '145023095187'
        KUBECONFIG_PATH = "/home/ec2-user/.kube/config"  // Use absolute path to kubeconfig
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

        stage('Ensure ECR Repositories Exist') {
            steps {
                script {
                    echo '🔹 Checking if AWS ECR repositories exist...'
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-credentials',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh '''
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY

                        aws ecr describe-repositories --repository-names user_management --region $AWS_REGION || \
                        aws ecr create-repository --repository-name user_management --region $AWS_REGION

                        aws ecr describe-repositories --repository-names order_management --region $AWS_REGION || \
                        aws ecr create-repository --repository-name order_management --region $AWS_REGION

                        aws ecr describe-repositories --repository-names postgres --region $AWS_REGION || \
                        aws ecr create-repository --repository-name postgres --region $AWS_REGION
                        '''
                    }
                }
            }
        }

//         stage('Run Ansible Playbook on EC2') {
//             steps {
//                 script {
//                     echo '🔹 Running Ansible Playbook to handle Docker build & push, and PostgreSQL setup...'
//                     dir('Ansible') {
//                         sh 'ansible-playbook -i inventory.ini complete_deployment.yml'
//                     }
//                 }
//             }
//         }

        stage('Setup AWS EKS Kubeconfig') {
            steps {
                script {
                    echo '🔹 Configuring Kubernetes access for AWS EKS on EC2...'
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-credentials',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh '''
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY

                        ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/my-key.pem ec2-user@51.20.115.71 << 'EOF'
                            echo '✅ Using existing kubeconfig for AWS EKS...'
                            export KUBECONFIG=/home/ec2-user/.kube/config
                            kubectl config current-context
                        EOF
                        '''
                    }
                }
            }
        }



        stage('Apply AWS ECR Secret in Kubernetes') {
            steps {
                script {
                    echo '🔹 Applying AWS ECR Kubernetes Secret on EC2...'
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/my-key.pem ec2-user@51.20.115.71 << 'EOF'
                        kubectl apply -f /home/ec2-user/springboot_sample_deployment/kubernetes/aws-ecr-secret.yml --kubeconfig /home/ec2-user/.kube/config
                    EOF
                    '''
                }
            }
        }

        stage('Apply Kubernetes Secrets & Deploy Applications') {
            steps {
                script {
                    echo '🔹 Deploying user and order management apps to AWS EKS from EC2...'
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/my-key.pem ec2-user@51.20.115.71 << 'EOF'
                        kubectl apply -f /home/ec2-user/springboot_sample_deployment/kubernetes/kubernetes-secrets.yml --kubeconfig /home/ec2-user/.kube/config
                        kubectl apply -f /home/ec2-user/springboot_sample_deployment/kubernetes/user_management-deployment.yml --kubeconfig /home/ec2-user/.kube/config
                        kubectl apply -f /home/ec2-user/springboot_sample_deployment/kubernetes/user_management-service.yml --kubeconfig /home/ec2-user/.kube/config
                        kubectl apply -f /home/ec2-user/springboot_sample_deployment/kubernetes/order_management-deployment.yml --kubeconfig /home/ec2-user/.kube/config
                        kubectl apply -f /home/ec2-user/springboot_sample_deployment/kubernetes/order_management-service.yml --kubeconfig /home/ec2-user/.kube/config
                    EOF
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo '🔹 Checking if all pods and services are running on EC2...'
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/my-key.pem ec2-user@51.20.115.71 << 'EOF'
                        kubectl get pods -o wide --kubeconfig /home/ec2-user/.kube/config
                        kubectl get svc -o wide --kubeconfig /home/ec2-user/.kube/config
                    EOF
                    '''
                }
            }
        }


        stage('Cleanup') {
            steps {
                script {
                    echo '🧹 Cleaning up unnecessary files on Jenkins and EC2...'

                    // Cleanup on Jenkins
                    sh '''
                    rm -rf user_management/target order_management/target
                    rm -f /var/lib/jenkins/tmp/kubernetes-secrets.yml
                    sudo docker system prune -a -f
                    '''

                    // Cleanup on EC2
                    echo '🧹 Cleaning up files on EC2 instance...'
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/my-key.pem ec2-user@51.20.115.71 << 'EOF'
                        sudo rm -rf /home/ec2-user/kubernetes/
                        sudo rm -rf /home/ec2-user/springboot_sample_deployment
                        sudo docker system prune -a -f
                        sudo rm -f /tmp/kubernetes-secrets.yml
                        sudo rm -rf /var/lib/docker/containers/*
                    EOF
                    '''

                    echo '✅ Cleanup completed successfully on Jenkins & EC2.'
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
