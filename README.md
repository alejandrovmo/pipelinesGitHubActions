# Pipelines with GitHub Actions
Pipelines with GitHub Actions


STEP#1 - Create Docker file for Node.js app
1. We create a docker file like this (This is an example):

	FROM node:14
	
	WORKDIR /usr/src/app
	
	COPY package.json .
	
	RUN npm install
	
	COPY . .
	
	EXPOSE 3000
	
	CMD ["node", "index.js"]

STEP#2 - Create a package.json file:

1. package.json

	{
    		"name":
    		"docker_nodejs_demo",
    		"version": "1.0.0",
    		"description": "",
    		"main": "index.js",
    		"scripts": {
        		"test": "echo \"Error: no test specified\" && exit 1"
    		},
    		"keywords": [],
    		"author": "",
    		"license": "ISC",
    		"dependencies": {
        		"config": "3.3.9",
        		"express": "4.18.2"
    			}
	}

STEP#3 - Create a repo en Docker Hub, and the repo secrets on GitHub for connecting GitHub, DockerHub and AWS:

1. Create the new repo in Docker Hub
2. Create  a acces token in Docker Hub -  Go to varmon - Account Settings - Security - New Acces token
3. Create secrets in Github, in your repo recently created:
	1. Go to ""SETTINGS"-Secrets and variables" - "Actions" -"NEW REPOSITORY SECRET"and create  the environment secrets.
	2. Create a Environment --> Dockerhub
		1. DOCKERHUB_PASSWORD ---> token created in dockerhub
		2. DOCKERHUB_USERNAME --> account used in DockerHub
	3. Upload files (Dockerfile, packages.json, etc) to the repo branch "main" or  where you  are going to create the workflow with github actions 

Step#4 - Github Actions workflow push Docker image to DockerHub using Github Actions:

1. Now configure GitHub Actions. For the image, building and pushing it to DockerHub and here we are going to use GitHub Actions:
 - Let's create a REPO for this pourpose (you can use a existing repo):
 - Go to Actions, after the repo creation and choose star a workflow for yourself.
 - Inside Edit new file, choose a name for the new yaml file, and insert the next lines of code:
		
      		name: Build and Push Docker image to Docker Hub
			on: 
			  push:
			     branches: 
			       - "main"
			 jobs:
			   push_to_registry:
			     name: Push Docker image to Docker Hub
			     runs-on: ubuntu-latest
			     steps:
			       - name: Check out the repo
			         uses: actions/checkout@v3
			       
			       - name: Login to Docker Hub
			         uses: docker/login-action@v2
			         with:
			           username: ${{ secrets.DOCKERHUB_USERNAME }}
			           password: ${{ secrets.DOCKERHUB_TOKEN }}
			       
			       - name: Extract metadata (tags, labels) for Docker
			         id: meta
			         uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
			         with:
			           images: varmon/pipelinegithubactions
			       
			       - name: Build and push Docker image
			         uses: docker/build-push-action@v4
			         with:
			           context: .
			           push: true
			           tags: varmon/pipelinegithubactions:latest

	Let’s understand our workflow first:
	  DOCKERHUB_USERNAME and DOCKERHUB_TOKEN are the variables needed to connect this job with docker hub container.
    
	Steps: Steps represent a sequence of tasks that will be executed as part of the job steps
  
	Job: Name: Check out code
	  It job simply checks out our GitHub repository for “Dockerfile” to build the docker image.
    
	Job: Name: Configure DockerHub credentials
	  In this job we need to configure AWS login Credentials. For accessing the DockerHub we need to define a custom Role in later steps.
    
	Job:Name: Build, tag, and push image to DockerHub
	  In this step we are going to build the Docker Image by copying using the Code in our Repository, tagging the Image with Version and pushing Image to DockerHub.
