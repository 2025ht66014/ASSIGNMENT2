pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "aceest/fitness-app"
        DOCKER_TAG = "${env.BRANCH_NAME == 'main' ? 'latest' : env.BRANCH_NAME}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").inside {
                        sh 'pytest'
                    }
                }
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                    sh """
                    docker stop fitness_app || true
                    docker rm fitness_app || true
                    docker run -d --name fitness_app -p 5000:5000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
    }
    post {
        failure {
            mail to: 'devteam@example.com',
                 subject: "Build Failed: ${env.JOB_NAME} for ${env.BRANCH_NAME}",
                 body: "Please check Jenkins logs for details."
        }
    }
}
