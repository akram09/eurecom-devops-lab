**NETSOFT                                                    
Adlen Ksentini**

## Introduction
This lab teaches you how to deploy a simple Flask web application in a Kubernetes cluster while incorporating linting, code formatting, and CI/CD automation using Jenkins. You’ll containerize the Flask application using Docker, deploy it on Kubernetes, and automate these processes with a Jenkins pipeline.

By the end of this lab, you’ll:

- **Deploy and containerize a Flask app.**
- **Apply linting and formatting with Flake8 and Black.**
- **Automate deployment using Jenkins pipelines.**
- **Monitor and troubleshoot Kubernetes deployments.**
## Prerequisites
Ensure you have the following installed:

**Docker**, **Kubernetes**, **kubectl**, **Minikube/KIND**, **Jenkins**, **Git**, and **Python 3.8+**.

## Lab Steps
### Section 1: Setting Up the Flask Application 
1. Create a Flask Application

    - Create a directory and Flask app:

    ```bash
    mkdir flask-app
    cd flask-app
    ```

    - Write the app.py file:

    ```python
    from flask import Flask
    
    app = Flask(__name__)
    
    @app.route("/")
    def home():
        return "Hello, Kubernetes!"
    
    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)
    ```

    - Create a requirements.txt file:
    ```text
    Flask==2.3.2
    ```
    
**Question 1: What is the purpose of requirements.txt?**

2. Test the Application Locally

    - Install dependencies and run Flask:

    ```bash
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
    python app.py
    ```
    
    - Access the app at http://127.0.0.1:5000.

**Question 2: What message do you see when accessing the app in your browser?**

3. Containerize the Application

    - Create a Dockerfile:

    ```Dockerfile
    FROM python:3.8-slim

    WORKDIR /app

    COPY requirements.txt .
    RUN pip install -r requirements.txt

    COPY . .

    CMD ["python", "app.py"]
    ```

    - Build and test the Docker image:

    ```bash
    docker build -t flask-app .
    docker run -p 5000:5000 flask-app
    ```

**Question 3: What does the docker run command do, and why is -p 5000:5000 necessary?**

### Section 2: Kubernetes Deployment 
Create Deployment and Service

Write deployment.yaml:

yaml
Copy
Edit
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: flask-app
        ports:
        - containerPort: 5000
Write service.yaml:

yaml
Copy
Edit
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
  type: LoadBalancer
Question 4: What is the difference between a Deployment and a Service in Kubernetes?

Deploy on Kubernetes

Apply the files:

bash
Copy
Edit
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
Verify deployments:

bash
Copy
Edit
kubectl get pods
kubectl get services
Question 5: What is the external IP of your service, and how do you access it?

Test the Application

Access the Flask app using the service’s external IP.

Question 6: What happens when you scale the deployment to 4 replicas?
(Hint: Use kubectl scale deployment flask-app --replicas=4.)

Section 3: Code Quality with Linting and Formatting (30 minutes)
Set Up Flake8 and Black

Install tools:

bash
Copy
Edit
pip install flake8 black
Run Flake8 for linting:

bash
Copy
Edit
flake8 app.py
Run Black for formatting:

bash
Copy
Edit
black app.py
Question 7: How does Black ensure consistent code formatting?

Automate Linting and Formatting

Install pre-commit hooks:

bash
Copy
Edit
pip install pre-commit
Create .pre-commit-config.yaml:

yaml
Copy
Edit
repos:
  - repo: https://github.com/psf/black
    rev: 23.1a1
    hooks:
      - id: black
  - repo: https://gitlab.com/pycqa/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
Install hooks:

bash
Copy
Edit
pre-commit install
Question 8: What happens when you commit code that doesn’t follow Black’s formatting rules?

Section 4: Automate Deployment with Jenkins (45 minutes)
Run Jenkins in Docker

Start Jenkins:

bash
Copy
Edit
docker run -d -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
Access Jenkins at http://localhost:8080.

Question 9: How do you retrieve the initial admin password for Jenkins?

Create a Jenkinsfile

Write a Jenkinsfile:

groovy
Copy
Edit
pipeline {
    agent any
    stages {
        stage('Lint') {
            steps {
                sh 'flake8 app.py'
            }
        }
        stage('Format') {
            steps {
                sh 'black --check app.py'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t flask-app .'
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
    }
}
Add the pipeline job in Jenkins.

Question 10: What stages are included in the pipeline, and what does each one do?

Trigger the Pipeline

Push the code to a Git repository and trigger the pipeline.

Question 11: How do you check the logs for each pipeline stage?

Lab Report Guidelines
Your report should include:

Summary: Summarize what you did in each section.
Answers to Questions: Provide answers to all the questions.
Screenshots/Logs: Include evidence of your deployment and testing.
Final Questions
Why is CI/CD important for modern software development?
What role do linting and formatting tools play in maintaining code quality?
How does Kubernetes simplify application scaling?
