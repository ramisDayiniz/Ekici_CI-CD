pipeline {
    agent any // Wir nutzen den Jenkins-Host, um Docker-Befehle zu steuern

    environment {
        IMAGE_NAME = "hellospencer-app"
        CONTAINER_NAME = "spencer-service"
        APP_PORT = "5556"
    }

    stages {
        stage('Checkout') {
            steps {
                // Code vom Repo holen
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Baue das Docker-Image...'
                // Erstellt das Image basierend auf deinem Dockerfile
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Test') {
            steps {
                echo 'Führe Unit-Tests im temporären Container aus...'
                // Startet einen Container nur für die Tests und löscht ihn danach (--rm)
                sh "docker run --rm ${IMAGE_NAME}:latest python -m pytest tests/"
            }
        }

        stage('Deployment') {
            steps {
                echo 'Bereite Deployment vor...'
                // Stoppe und entferne alte Container mit diesem Namen, falls sie existieren
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"

                echo "Starte Applikation auf Port ${APP_PORT}..."
                // Startet den finalen Container
                sh "docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:5556 ${IMAGE_NAME}:latest"
            }
        }

        stage('Verify') {
            steps {
                echo 'Überprüfe API...'
                sleep 5 // Wartezeit für Flask-Start
                sh "curl http://localhost:${APP_PORT}/api/hello || echo 'API noch nicht bereit'"
            }
        }
    }
}