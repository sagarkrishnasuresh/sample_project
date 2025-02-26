pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
        MINIKUBE_HOME = '/var/lib/jenkins/.minikube'
        PATH = '/usr/local/bin:/usr/bin:/bin'
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

        stage('Setup Temporary Directory') {
            steps {
                script {
                    sh '''
                    if [ ! -d "/var/lib/jenkins/tmp" ]; then
                        mkdir -p /var/lib/jenkins/tmp
                        echo "Directory /var/lib/jenkins/tmp created."
                    else
                        echo "Directory /var/lib/jenkins/tmp already exists."
                    fi
                    '''
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                dir('Ansible') {
                    sh 'ansible-playbook complete_deployment.yml'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                dir('kubernetes') {
//                     sh 'kubectl apply -f postgres-deployment.yml'
                    sh 'kubectl apply -f postgres-service.yml'
                    sh 'kubectl apply -f user_management-deployment.yml'
                    sh 'kubectl apply -f user_management-service.yml'
                    sh 'kubectl apply -f order_management-deployment.yml'
                    sh 'kubectl apply -f order_management-service.yml'
                }
            }
        }
        stage('Clearing residuals') {
            steps {
                script {
                    echo '🔄 Clearing non-needed files after deployment...'

                    // Clear Maven target directories
                    sh 'rm -rf user_management/target'
                    sh 'rm -rf order_management/target'

                    // // Clear Ansible temporary files
                    // sh 'rm -f Ansible/roles/postgres_setup/templates/kubernetes-secrets.yml'
                    // sh 'rm -f /var/lib/jenkins/tmp/kubernetes-secrets.yml'

                    

                    // Clear Docker build cache
                    sh 'sudo docker system prune -a -f'

                    // Clear Kubernetes logs (optional)
                    sh 'rm -rf ~/.kube/cache'
                    sh 'rm -rf ~/.kube/http-cache'

                    echo '✅ Residual files cleared successfully.'
                }
            }
        }
    }
    post {
        success {
            echo 'Build and deployment completed successfully!'
        }
        failure {
            echo 'Build or deployment failed. Check logs for details.'
        }
    }
}
