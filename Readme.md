🧪 Experiment: CI/CD Pipeline using Git, Jenkins and Docker (Port 9084)
🎯 Aim

To implement a CI/CD pipeline where code pushed to GitHub triggers Jenkins Pipeline to build a Docker image and deploy a Flask web application (with UI) on port 9084.

🧰 System Requirements
OS: Windows / Linux / macOS
Internet connection
Minimum 4GB RAM
⚙️ STEP 1: Install Required Tools
🔹 Install Git
Windows:

Install Git → Next → Finish

🔹 Install Docker

Windows / Mac:

Install Docker Desktop

🔹 Verify Docker
docker --version

🔹 Stop old container (if exists)

docker stop jenkins
docker rm jenkins

🔹 Run Jenkins (Port 9084)

docker run -d --name jenkins -p 9084:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v //var/run/docker.sock:/var/run/docker.sock -u root jenkins/jenkins:lts

🔹 Open Jenkins
http://localhost:9084

🔹 Unlock Jenkins

docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

✔ Install suggested plugins
✔ Create admin user

⚙️ STEP 2: Give Jenkins Access to Docker

docker exec -it jenkins bash -c "apt-get update && apt-get install -y docker.io"

docker exec -it jenkins docker --version

docker exec -u root jenkins groupadd docker

docker exec -u root jenkins usermod -aG docker jenkins

docker restart jenkins

🌐 STEP 3: Create Flask Application (With HTML UI)

ci-cd-app
cd ci-cd-app


🔹 app.py

from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=9080)

🔹 templates/index.html

<!DOCTYPE html>
<html>
<head>
    <title>CI/CD Pipeline</title>
    <style>
        body { font-family: Arial; text-align: center; margin-top: 50px; background:#f2f2f2; }
        .box { background:white; padding:20px; border-radius:10px; width:350px; margin:auto; box-shadow:0 0 10px gray; }
    </style>
</head>
<body>
<div class="box">
    <h1>🚀 CI/CD Pipeline</h1>
    <p>Deployed using Jenkins Pipeline + Docker</p>
    <p><b>Running on Port 9084</b></p>
</div>
</body>
</html>

🔹 Create requirements.txt

flask==2.3.2

📦 STEP 4: Create Dockerfile

FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 9080
CMD ["python", "app.py"]


📄 STEP 5: Create Jenkinsfile (Pipeline Automation)

pipeline {
    agent any

    environment {
        IMAGE_NAME = "ci-cd-app"
        CONTAINER_NAME = "ci-cd-container"
        PORT = "9080"
    }

    stages {

        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Remove Old Container') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p ${PORT}:${PORT} \
                    ${IMAGE_NAME}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ App deployed at http://localhost:${PORT}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}

☁️ STEP 6: Push Code to GitHub

Go to GitHub

git init
git add .
git commit -m "Initial commit with UI + Jenkinsfile"
git branch -M main
git remote add origin https://github.com/<username>/ci-cd-app.git
git push -u origin main

⚙️ STEP 7: Create Jenkins Pipeline Job

🔹 Create Job
New Item → Pipeline
Name: CI-CD-Pipeline

🔹 Configure

Definition → Pipeline script from SCM
SCM → Git
Repository URL → your repo

Change master To main In Branch

🔹 Enable Trigger

✔ GitHub hook trigger for GITScm polling

🔹 Save

Then Add Environment Varibles In Jenkins
Go To Setting
In That System
Scroll Down
And Then Tick Environment Variables And Click Add

PATH

/usr/bin:/usr/local/bin:/bin:/usr/sbin:/sbin

🌐 STEP 8: Setup ngrok (Webhook Access)

🔹 Install ngrok
https://ngrok.com/download/windows?tab=download

Windows:
Download ZIP → Extract → move to C:\ngrok
Add to PATH

🔹 Add Auth Token Go To ngrok.com And Login/Signup And Then Go To Your Authtoken Copy And Enter In VS Code Terminal If Error Close Folder And Quit VS Code And Reopen

https://dashboard.ngrok.com/signup

Example:
ngrok config add-authtoken YOUR_AUTH_TOKEN

🔹 Start ngrok

ngrok http 9084

🔹 Copy URL

Example:

https://abc123.ngrok-free.app

🔹 Add Webhook in GitHub

Payload URL:

https://abc123.ngrok-free.app/github-webhook/


▶️ STEP 9: Trigger Pipeline

Change In index.html
Add Line:
<p>Hello World</p>

Run The Commands

git add .
git commit -m "Trigger CI/CD"
git push

🌍 STEP 10: Verify Output

Open:

http://localhost:9080

✅ Output
Web UI page displayed
Pipeline executed automatically