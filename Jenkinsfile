pipeline {
    agent any

    environment {
        // Jenkins credentials ID for Docker Hub (configure in Jenkins)
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        // Full image name on Docker Hub – change this to your own
        IMAGE_NAME = 'your-dockerhub-username/sum-product_fx'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven JAR') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn clean package -DskipTests'
                    } else {
                        bat 'mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def cmd = "docker build -t ${env.IMAGE_NAME}:latest ."
                    if (isUnix()) {
                        sh cmd
                    } else {
                        bat cmd
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.DOCKERHUB_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        if (isUnix()) {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                docker push "$IMAGE_NAME:latest"
                            '''
                        } else {
                            bat """
                                docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                                docker push %IMAGE_NAME%:latest
                            """
                        }
                    }
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
