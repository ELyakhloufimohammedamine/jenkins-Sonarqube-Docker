pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mywebsite:latest'
        DOCKER_REPO = 'jenkins-Sonarqube-Docker/mywebsite'
        AWS_EC2_IP = '54.163.4.0'  
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ELyakhloufimohammedamine/jenkins-Sonarqube-Docker.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker tag $DOCKER_IMAGE $DOCKER_REPO'
                sh 'docker push $DOCKER_REPO'
            }
        }

        stage('Deploy to AWS') {
            steps {
                sshagent(['jenkins-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@$AWS_EC2_IP << EOF
                        docker pull $DOCKER_REPO
                        docker stop mon-projet || true
                        docker rm mon-projet || true
                        docker run -d --name mon-projet -p 80:80 $DOCKER_REPO
                    EOF
                    '''
                }
            }
        }
    }
}
