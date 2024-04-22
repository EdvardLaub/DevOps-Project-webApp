# example-jenkins-pipeline

Sample Jenkins pipeline example. The pipeline is supposed to be used with MultiBranch Pipeline jobs.

## Sample Web-App

The sample NodeJS based web-app, courtesy of testdriven.io, includes the following essential areas to demonstrate a typical CI/CD pipeline:
 
 - Static Analysis
 - Unit Testing, and
 - Code Coverage

## Jenkins Playground

The folder `jenkins-playground` in this repository contains docker-compose based orchestration for a fully functional containerized Jenkins server with 1 Linux agent. 
In addition there is a Nginx based test node container spinned up for testing deployment to the staging environment.

### Host requirements

The Jenkins playground can run on any Linux or Windows Linux Subsystem (WSL) based environment. The following additional items are required on the chosen Linux host:

 - docker
 - docker-compose
 - The `/data/jenkins` persistent folder with Read/Write/Execution permissions for the world.

### Setup Instructions

 - Generate a single private/public key pair for jenkins agent and test node.
  - You will protect the private key using Jenkins Credentials feature (or any other secrets manager).
 - Set the public key string in the `docker-compose.yml` file in the following environment variables
   - JENKINS_AGENT_SSH_PUBKEY
     - This public key is used to communicate with the agent to start it up
   - SSH_AUTHORIZED_KEY
     - This public key is used to deploy the web-app to the test node via SCP.

### Running Jenkins Playground Environment

``` bash
cd jenkins-playground
docker-compose up --build --detach
```

You can also use Visual Studio Code to contol the Jenkins Playground environment via the docker compose extension:
 - right click on the `docker-compose.yml` file
 - select "Compose Up"

The Jenkins playground is accesible via http://localhost:8080 on the host

NOTE: You will need to grab the initial administrator token from the Jenkins logs, either using docker log or examining
the logs in the persistent volume folder `/data/jenkins`

### Additional Jenkins Plugins

Select the recommended typical plugins when prompted for the initial plugins. In addition, once the system is setup install the following additional Jenkins plugins:

 - nodejs: For supporting installing NodeJS tool environment.
   - Add the referenced installation id `Node 20.x` to NodeJS tools configuration. 
   - Select the latest stable NodeJS version from the v20.x LTS channel.
 - clover: For reporting code coverage.

### Additional Setup

Using Jenkins credential feature, you will need to setup SSH private key for the following credential ids used in the pipeline:

 - `jenkins-app-staging`: The private key for the test node that the application will be deployed to in the staging environment.

The Jenkins credential feature is sufficient to store these secrets. Refer to Jenkins guides for this. 

## MultiBranch Pipeline Parameters

Setup a MultiBranch Pipeline, connect the SCM to this GitHub repository, and reference the Jenkinsfile from this repository. The following parameters are available for the pipeline:

  - NODEJS_VERSION: The version of NodeJS to use.
  - DEPLOY_USER: The user for deploying application to the test node.
  - DEPLOY_NODE: The node to deploy the application to the test node.

## Test Node

The Jenkins playground has a test node available to deploy the sample web-app. You can use this node or any other node. If you choose to use
your own test node, override the Jenkins jobs parameters DEPLOY_USER and DEPLOY_NODE for your test node and set the SSH private key for the
credential id `jenkins-app-staging` for your test node. The application will be deployed to the `/public` folder on the test node.

## Sample Test Environment

If you used the test node provided by the Jenkins playground, then the sample web-app is accessible via http://localhost:9080 on host environment.