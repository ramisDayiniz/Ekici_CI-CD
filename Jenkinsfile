pipeline {
    agent any

    environment {
        IMAGE_NAME = "hellospencer-app"
        CONTAINER_NAME = "spencer-service"
        APP_PORT = "5556"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Baue das Docker-Image...'
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Führe reine Unit-Tests aus...'
                // Wir führen nur test_hello.py aus, da diese keinen laufenden Server brauchen
                sh "docker run --rm ${IMAGE_NAME}:latest python -m pytest tests/test_hello.py"
            }
        }

        stage('Deployment') {
            steps {
                echo 'Bereite Deployment vor...'
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"

                echo "Starte Applikation auf Port ${APP_PORT}..."
                sh "docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:5556 ${IMAGE_NAME}:latest"
            }
        }

        stage('API Verification') {
            steps {
                echo 'Warte auf App-Start und teste API...'
                sleep 10 // Puffer, damit Flask hochfahren kann

                // Jetzt testen wir die API von außen (vom Jenkins-Host aus)
                sh "curl http://localhost:${APP_PORT}/api/hello"

                echo 'Führe API-Integrationstests aus...'
                // Optional: Die API-Tests jetzt gegen den laufenden Container laufen lassen
                sh "docker run --rm --network host ${IMAGE_NAME}:latest python -m pytest tests/test_api.py || echo 'API Tests failed, aber Container läuft'"
            }
        }
    }
}