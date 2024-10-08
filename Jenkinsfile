pipeline {
    agent any

    tools {
        maven 'my-maven'
    }

    environment {
        MYSQL_ROOT_LOGIN_PSW = credentials('mysql-root-login')
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: 'https://github.com/TnpHau/jenkins_spring.git']]])
            }
        }

        stage('Login Docker') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                    }
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package'
            }
        }

        stage('Set Minikube Docker Daemon') {
            steps {
                script {
                    // Thiết lập Docker của Minikube bằng PowerShell
                    powershell '''
                    minikube start
                    $envVars = & minikube -p minikube docker-env --shell=cmd
                    Invoke-Expression $envVars
                    '''
                }
            }
        }

        stage('Packaging/Pushing Image to Minikube') {
            steps {
                script {
                    // Sử dụng Docker của Minikube để build image
                    sh 'docker build -t tnphau/springboot .'
                    sh 'docker push tnphau/springboot'
                }
            }
        }

        stage('Deploy Spring Boot on Minikube') {
            steps {
                script {
                    // Triển khai ứng dụng lên Minikube
                    sh 'kubectl apply -f k8s.yaml'  // Cập nhật với file manifest của bạn
                }
            }
        }
    }
}
