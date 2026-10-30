pipeline {
    agent any

    triggers {
        // Poll Git for new commits approximately every two minutes.
        pollSCM('H/2 * * * *')
    }

    environment {
        APP_IMAGE = 'student-flask-cicd'
        CONTAINER_NAME = 'student-flask-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Test Image') {
            steps {
                sh 'docker build --target test -t ${APP_IMAGE}-test:${BUILD_NUMBER} .'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'docker run --rm ${APP_IMAGE}-test:${BUILD_NUMBER}'
            }
        }

        stage('Build Application Image') {
            steps {
                sh 'docker build --target runtime -t ${APP_IMAGE}:${BUILD_NUMBER} -t ${APP_IMAGE}:latest .'
            }
        }

        stage('Deploy Locally') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true
                    docker run -d --restart unless-stopped \\
                      --name ${CONTAINER_NAME} \\
                      -p 8081:5000 \\
                      ${APP_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deployment Check') {
            steps {
                sh '''
                    sleep 3
                    docker ps --filter "name=${CONTAINER_NAME}"
                    docker exec ${CONTAINER_NAME} python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:5000/health').read().decode())"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD completed: tests passed and the application is deployed.'
            echo 'Open http://localhost:8081 in your browser.'
        }
        failure {
            echo 'CI/CD pipeline failed. Check the stage logs above.'
        }
    }
}
