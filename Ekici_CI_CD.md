# Middleware Engineering "CI/CD Pipelines in Jenkins"

##### Verfasser: Ramis Ekici

##### Datum: 11.05.2026

# Einführung

In dieser Aufgabe wird CI CD dokumentiert. Wir erstellen ein CI CD Pipeline und lernen die die Umsetzung mit Jenkins.

# Vorbereitung

- Wie immer Github Repo kopieren...

- Jenkins Container erstellen

```docker
docker run -u root -d \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name jenkins-server \
  jenkins/jenkins:latest
```

- Nachdem Container gestartet wurde diesen Befehl eingeben, um das Passwort zu erfahren:

```docker
docker logs jenkins-server
```

- Später auf unter localhost:8080 das Passwort einfügen

- Dann suggested Plugins installieren.

- Danach einen Admin User erstellen.

- Danach auf Jenkins starten drücken

- Anschließend brauchen wir noch den Docker CLI mit diesem Befehl:

```docker
docker exec -it -u root jenkins-server rm /etc/apt/sources.list.d/docker.list


docker exec -it -u root jenkins-server bash -c "apt-get update && apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && echo 'deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable' | tee /etc/apt/sources.list.d/docker.list > /dev/null && apt-get update && apt-get install -y docker-ce-cli"
```



# Praxis

### Plugins

Bevor wir mit der Aufgabe noch beginnen brauchen wir weitere Plugins, die nicht installiert wurden. Diese installieren wir unter Einstellungen unter Plugins:

- **Docker**

- **Docker Pipeline**

- **CloudBees Docker Build and Publish** 

### Jenkinfile

Das ist sozusagen das Gehirn oder Rezept der Automatisierung.

```textile
pipeline {
    agent any // Wir nutzen den Jenkins-Host direkt, da wir Docker-Befehle senden

    environment {
        IMAGE_NAME = "hellospencer-app"
        CONTAINER_NAME = "spencer-service"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Baue Docker Image...'
                sh "docker build -t ${IMAGE_NAME}:latest ."

                echo 'Führe Unit-Tests im Container aus...'
                sh "docker run --rm ${IMAGE_NAME}:latest python -m pytest tests/"
            }
        }

        stage('Deployment') {
            steps {
                echo 'Deploye auf lokales Notebook...'
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"

                sh "docker run -d --name ${CONTAINER_NAME} -p 5556:5556 ${IMAGE_NAME}:latest"
            }
        }

        stage('Verify') {
            steps {
                echo 'Überprüfe API Endpunkt...'
                sleep 5
                sh "curl http://localhost:5556/api/hello || echo 'API noch nicht bereit'"
            }
        }
    }
}
```

Danach ein neuen Pipeline auf Jenkins erstellen.

- Scrolle im Job ganz nach unten zum Abschnitt **Pipeline**.

- Ändere *Definition* auf **Pipeline script from SCM**.

- Wähle bei *SCM* den Eintrag **Git** aus.

- Gib unter **Repository URL** deinen Link ein: 

- Prüfe den **Branch Specifier**: Wenn dein Code auf dem Hauptzweig liegt, muss dort `*/main` (oder `*/master`) stehen.

- Stelle sicher, dass bei **Script Path** das Wort `Jenkinsfile` steht.

- Klicke auf **Save**.




