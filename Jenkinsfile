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
                    sh 'kubectl apply -f postgres-deployment.yml'
                    sh 'kubectl apply -f postgres-service.yml'
                    sh 'kubectl apply -f user_management-deployment.yml'
                    sh 'kubectl apply -f order_management-deployment.yml'
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
