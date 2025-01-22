**NETSOFT                                                    
Adlen Ksentini**

## Introduction
This lab teaches you how to deploy a simple Flask web application in a Kubernetes cluster while incorporating linting, code formatting, and CI/CD automation using Jenkins. You’ll containerize the Flask application using Docker, deploy it on Kubernetes, and automate these processes with a Jenkins pipeline.

By the end of this lab, you’ll:

- **Deploy and containerize a Flask app.**
- **Apply linting and formatting with Flake8 and Black.**
- **Automate deployment using Jenkins pipelines.**
- **Monitor and troubleshoot Kubernetes deployments.**

## Important ! 
We will be using the same environment as the last lab, so please use the same machine in order to access the Kubernetes cluster you have created previously.

## Prerequisites
Ensure you have the following installed:

**Docker**, **Kubernetes**, **kubectl**, **Minikube/KIND**, **Git**, and **Python 3.8+**.

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
        app.run(host="0.0.0.0", port=8080)
    ```

    - Create a requirements.txt file:
    ```text
    Flask==3.1.0
    ```
    
**Question 1: What is the purpose of requirements.txt?**

2. Test the Application Locally

    - Install dependencies and run Flask:

    ```bash
    python3 -m venv venv # Or use virtualenv  
    source venv/bin/activate
    pip install -r requirements.txt
    python app.py
    ```
    
    - Access the app at http://127.0.0.1:8080.

**Question 2: What message do you see when accessing the app in your browser?**

3. Containerize the Application

    - Create a Dockerfile:

    ```Dockerfile
    FROM python:3.9-slim

    WORKDIR /app

    COPY requirements.txt .
    RUN pip install -r requirements.txt

    COPY . .

    CMD ["python", "app.py"]
    ```

    - Build and test the Docker image:

    ```bash
    docker build -t flask-app .
    docker run -p 8080:8080 flask-app
    ```

**Question 3: What does the docker run command do, and why is -p 8080:8080 necessary?**

4. Pushing created image
   
    - Create a dockerhub account at https://hub.docker.com. The dockerhub registry allows for unlimited public repositories and one single private repository. 

    - Once the account created, run login on your machine. This will open a browser and sync with docker in your machine (we will clean everything later)
    
    ```bash
    docker login
    ```

    - Retag the created image to your actual dockerhub repository, then push to dockerhub.
    ```bash 
    docker tag flask-app:latest <dockerhub-username>/<repo-name>:0.0.1
    docker push <dockerhub-username>/<repo-name>:0.0.1
    ```

### Section 2: Kubernetes Deployment 
1. Create Deployment and Service

    - Write deployment.yaml:

    ```yaml
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
            image: <dockerhub-username>/<repo-name>:0.0.1
            ports:
            - containerPort: 8080
    ```
    
    - Write service.yaml:
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: flask-app-service
    spec:
      selector:
        app: flask-app
      ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
      type: ClusterIP
    ```
**Question 4: What is the difference between a Deployment and a Service in Kubernetes?**

2. Deploy on Kubernetes

    - Apply the files:

    ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

    - Verify deployments:

    ```bash
    kubectl get pods
    kubectl get services
    ```
    
**Question 5: What is the external IP of your service, and how do you access it?**
    
    - Test the Application

Access the Flask app using the port-forward created by kubernetes.

**Question 6: What happens when you scale the deployment to 4 replicas?
(Hint: Use kubectl scale deployment flask-app --replicas=4.)**

### Section 3: Code Quality with Linting and Formatting (30 minutes)

1. Set Up Flake8 and Black

    - Install tools:

    ```bash
    pip install flake8 black
    ```
    
    - Run Flake8 for linting:

    ```bash
    flake8 app.py
    ```
    - Run Black for formatting:

    ```bash
    black app.py
    ```

    

2. Automate Linting and Formatting
   

    - Setup a local git repository

    ```bash
    git init
    ```

    - Create .gitignore
    ```bash
    venv
    ```

    - Install pre-commit hooks:

    ```bash
    pip install pre-commit
    ```
    - Create .pre-commit-config.yaml:

    ```yaml
    repos:
      - repo: https://github.com/psf/black
        rev: 23.1a1
        hooks:
          - id: black
      - repo: https://github.com/pycqa/flake8
        rev: 6.0.0
        hooks:
          - id: flake8
    ```
    - Install hooks:

    ```bash
    pre-commit install
    ```

    - **Create a github repository and push the local repository.**
  
    - Make sure that pre-commit are running. Try to change in `app.py` to make the formatting showing error on pre-commit.
  
    - Take a screenshot of both a success push and an errored one.
    
**Question 8: What happens when you commit code that doesn’t follow formatting/linting rules?**

### Section 4: Automate Deployment with Jenkins 
1. Install Jenkins

    - Install using package manager:

    ```bash
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
      https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
      https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    ```
    
    - Start Jenkins
    ```bash
    systemctl start jenkins
    ```

    - Access Jenkins at http://localhost:8080.

1. **If you can't run Jenkins installation run it using docker**
   ```bash
   docker network create jenkins
   docker run \
      --name jenkins-docker \
      --rm \
      --detach \
      --privileged \
      --network jenkins \
      --network-alias docker \
      --env DOCKER_TLS_CERTDIR=/certs \
      --volume jenkins-docker-certs:/certs/client \
      --volume jenkins-data:/var/jenkins_home \
      --publish 2376:2376 \
      docker:dind \
      --storage-driver overlay2
   ```
    - Create a Jenkins dockerfile
   ```bash
    FROM jenkins/jenkins:2.479.3-jdk17
    USER root
    RUN apt-get update && apt-get install -y lsb-release
    RUN apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev -y 
    RUN apt install python3 python3-venv -y
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg
    RUN echo "deb [arch=$(dpkg --print-architecture) \
      signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
      https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    RUN apt-get update && apt-get install -y docker-ce-cli
    USER jenkins
    RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
   ```
    - Build it
   ```bash
   docker build -t myjenkins-blueocean:2.479.3-1 .
   ```
    - Run it
   ```bash
   docker run \
      --name jenkins-blueocean \
      --restart=on-failure \
      --detach \
      --network jenkins \
      --env DOCKER_HOST=tcp://docker:2376 \
      --env DOCKER_CERT_PATH=/certs/client \
      --env DOCKER_TLS_VERIFY=1 \
      --publish 8080:8080 \
      --publish 50000:50000 \
      --volume jenkins-data:/var/jenkins_home \
      --volume jenkins-docker-certs:/certs/client:ro \
      myjenkins-blueocean:2.479.3-1
   ```

**Question 9: How do you retrieve the initial admin password for Jenkins?**

2. Create a Jenkinsfile

    - Write a Jenkinsfile:

    ```groovy
    pipeline {
        agent any 
        stages {
            stage('build') {
                steps {
                    sh 'python3 --version'
                }
            }
        }
    }
    ```
3. Configure Jenkins to checkout github repository
    - Go To Jenkins dashboard and create a new item, Select a Pipeline and choose a name for the pipeline.
    - In general page select the github project checkbox and add the github repository link (Follow this structure https://github.com/<usernam>/<reponame>)
    - Go to Pipeline Section, in the definition item select Pipeline from SCM, and select GIt as the SCM. Finally set the name of the git branch you are using. 
    - You can try running the build by going back to the pipeline overview and click on Run Now.

**Question 10: What stages are included in the pipeline, and what does each one do?**

4. Setup of CI CD pipeline

   - Update your Jenkinsfile with the following content
  
   ```groovy
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
                    sh 'docker build -t <dockerhub-username>/<repo-name>:0.0.1 .'
                    sh 'docker push <dockerhub-username>/<repo-name>:0.0.1'
                }
            }
        }
    }
    ```
   - Trigger the pipeline. This pipeline won't execute well, since we need to make some adjustments on the pipeline and the machine we are executing the pipeline into.
   - First, to fix the issue with 'Lint' and 'Format' we can either install the flake8 and black dependencies on our system. Or install them inside the pipeline as follows:
   ```bash
   steps {
         sh '''
         #!/bin/bash
         python3 -m venv venv
         . ./venv/bin/activate
         pip install flake8
         flake8 app.py
         '''
     }
   ```
   - For docker configureation, we need to give the jenkins user permission to run docker
   ```bash
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   ```


   - We would also need to give `docker push` our credentials to login. For that we will create a Jenkins credentials and use it from the Jenkfinsfile.
   - Go To Jenkins Dashboard, in main page go to **Manage Jenkins** -> **Credentials** -> **System** -> **Global credentials (unrestricted)** -> **Add Credentials**
   - Here you have two options, either you specify your actual dockerhub username and password, or you generate a Personal Access Token from dockerhub and use it as the password.
   - Once the credential configured, update the Jenkinsfile accordingly:
   ```bash
   pipeline {
     agent any
     environment {
         DOCKER_CREDENTIALS = credentials('dockerhub-credentials') // Use the ID of your credentials
     }
   ```

   ```bash
   stage('Build') {
      steps {
         sh 'echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin'
      }
   }
   ```

   - Run the Pipeline.

4. Setup of Kubernetes Deployment Step
   - Now add a final step to deploy the built image to Kubernetes. **Please add the steps you have followed in the report**
   **PS:** You can test if the jenkins user can run kubectl commands by switching to that user on your machine
     ```bash
     sudo su - jenkins
     ```
   Make sure that kubeconfig is configured so that the jenkins user can run kubectl commands


