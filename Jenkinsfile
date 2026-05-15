pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk17'
    }

    environment {
        IMAGE_NAME = "CKShetty4/demo-app"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/CKShetty4/demo-app.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarServer') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=demo-app \
                        -Dsonar.token=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker rm -f demo-app-container || true
                docker run -d -p 8081:8081 --name demo-app-container $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
