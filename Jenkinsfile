pipeline {
    agent any

    tools {
        maven 'my-maven'
    }

    environment {
        MYSQL_ROOT_LOGIN_PSW = credentials('mysql-root-login')
    }

    stages {

//         stage('Checkout Code') {
//             steps {
//                 checkout([$class: 'GitSCM',
//                           branches: [[name: '*/main']],
//                           userRemoteConfigs: [[url: 'https://github.com/TnpHau/jenkins_spring.git']]])
//             }
//         }

        stage('Login Docker') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                    }
                }
            }
        }

//         stage('Deploy MySQL to DEV') {
//             steps {
//                 script {
//                     echo 'Deploying and cleaning'
//                     sh 'docker image pull mysql:latest'
//                     sh 'docker network create dev || echo "this network exists"'
//                     sh 'docker container stop tnphau-mysql || echo "this container does not exist"'
//                     sh 'echo y | docker container prune'
//                     sh 'docker volume rm tnphau-mysql-data || echo "no volume"'
//
//                     sh "docker run --name tnphau-mysql --rm --network dev -p 3306:3306 -v tnphau-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_LOGIN_PSW -e MYSQL_DATABASE=db_example -e MYSQL_USER=tnphau -e MYSQL_PASSWORD=Aa@123 -d mysql:latest"
//                     sh 'sleep 20'
//                 }
//             }
//         }

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package'
            }
        }

        stage('Packaging/Pushing Image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t tnphau/springboot .'
                    sh 'docker push tnphau/springboot'
                }
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                script {
                    echo 'Deploying and cleaning'
                    sh 'docker image pull tnphau/springboot'
                    sh 'docker container stop tnphau-springboot || echo "this container does not exist"'
                    sh 'docker network create dev || echo "this network exists"'
                    sh 'echo y | docker container prune'

                    sh 'docker container run -d --rm --name tnphau-springboot -p 8081:8080 --network dev tnphau/springboot'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
