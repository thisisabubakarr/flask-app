pipeline {
    agent any

    environment {
        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'flask-redis-app'
        IMAGE_TAG = 'latest'
        BLUE_PORT = '8000'
        GREEN_PORT = '8001'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SalmanS-19/flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("http://${env.REGISTRY}") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh '''
                cd /var/lib/jenkins/workspace/Flask-App-Blue-Green/BlueGreen
                docker-compose -f docker-compose-green.yml pull
                docker-compose -f docker-compose-green.yml up -d web_green
                '''
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                echo "Access the Green environment at http://localhost:${env.GREEN_PORT}"
            }
        }

        stage('Test Green Environment') {
            steps {
                script {
                    def response = sh(script: "curl -o /dev/null -s -w '%{http_code}' http://localhost:${env.GREEN_PORT}", returnStdout: true).trim()
                    if (response == '200') {
                        echo "Green environment is up and running with HTTP 200 response."
                    } else {
                        error "Green environment did not return HTTP 200. Response code: ${response}"
                    }
                }
            }
        }

        stage('Switch Traffic to Blue if Tests Fail') {
            when {
                expression { currentBuild.result == 'FAILURE' }
            }
            steps {
                echo "Rollback by continuing to use Blue environment on port ${env.BLUE_PORT}"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
