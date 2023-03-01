# Jenkins CD/CD Project

In this project, we will dockerize a React app and push it to Dockerhub registry.

## Setting up Jenkins in Docker to run Docker

This most tricky part of this project is to properly configure Jenkins container so you can run docker commands in the container to build/push images.

This goal is achieved by creating a custom jenkins docker image and install docker and mount host machine volum to leverage docker deamon.

We can create our image using the below **Dockerfile**, installing all necessary dependencies and add jenkins user to Docker group.

```
FROM jenkins/jenkins:lts
USER root
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get -y install docker-ce
RUN usermod -aG docker jenkins

```

Build the image with `docker image build -t custom-jenkins-docker .`.

Run the container with `docker run -it -p 8080:8080 -p 50000:50000 -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home custom-jenkins-docker` to use volume and connect the Docker CLI in the Jenkins container to the Docker daemon on the host machine.

## Add your docker hub credentials

To connect to Dockerhub programmatically, you need to use an **Access Tocken**. It can be generated from https://hub.docker.com/settings/security.

After getting you **Access Token**, create a global username/password type credential with it. The username should be your Dockerhub username.

## Create a Dockfile to dockerize your React app

Dockerize your app with the below sample docker file.

```docker
# Dockerfile

# =======Configuration=======
# Use a node 16 base image
FROM node:16-alpine
# Set the working directory to /app inside the container
WORKDIR /app
# Copy app files
COPY . .

# =========Build==========
# Install dependencies (npm ci makes sure the exact versions in the lockfile gets installed)
RUN npm ci
# Build the app
RUN npm run build

# ==========Run===========
# Set the env to "production"
ENV NODE_ENV=production
# Expose the port on which the app will be running (3000 is the default that `serve` uses)
EXPOSE 3000
# Start the app
CMD [ "npx", "serve", "build" ]
```

## Create a pipeline with Jenkins file

Create a Jenkinsfile workflow file like below to build your image and push to DockerHub.

```
pipeline {
	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('YOUR_CREDENTIAL_ID')
	}

	stages {
		stage('Build') {
			steps {
				sh 'docker build -t USERNAME/IMAGE_NAME:TAG .'
			}
		}
		stage('Login') {
			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
		stage('Push') {
			steps {
				sh 'docker push USERNAME/IMAGE_NAME:TAG'
			}
		}
	}

	post {
		always {
			sh 'docker logout'
		}
	}
}
```
