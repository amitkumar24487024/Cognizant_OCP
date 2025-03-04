## Harness CI/CD Lab: Deploying WebApp to OpenShift

### Task 1: Access Harness Platform

* Go to the Harness registration page and register for an account if you haven't already.

* Log in to the Harness web UI. *https://app.harness.io/auth/#/signin*
  

### Task 2: Create a New Project

* Navigate to Projects and click + New Project.

* Enter the project name: *demo-project*

* Click Save and Continue.

### Task 3: Set Up Connectors

#### 1. GitHub Connector:

* Go to Connectors under Project Settings.

* Click + New Connector and choose GitHub.

* Set the following:

    * URL Type: *Account*
    
    * Connection Type: *HTTP*
    
    * GitHub Account URL: *https://github.com/sirinali07/*
    
    * Repository Name: *git-repo* (Use your YAML repo name)

* Click Continue.

* Authenticate and validate the connection.

* Provide your GitHub username and add a Personal Access Token in Secret, and Click Continue.

* Select **Connect to the Provider** and click Save and Continue.

* Complete the Connection Test and click Finish.

#### 2. DockerHub Connector

* Go to Connectors and click + New Connector.

* Select Docker Registry.

* Provide the name: *my-docker-registry*. Click Continue.

* Enter DockerHub credentials:

    * Docker Registry URL: *https://index.docker.io/v1/*
    
    * Username: *ocpctregistry*
    
    * Password: Configure secret <docker-secret>

* Click Continue.

* Select Connect to the Provider and click Save and Continue.

Complete the Connection Test and click Finish.

#### 3. OpenShift/Kubernetes Connector

* Click + New Connector and select Kubernetes Cluster.

* Provide the name: *my-ocp-cluster*, Click Continue.

* Choose *Use the credentials of a specific Harness Delegate*, Click Continue.

* If there is no pre-installed delegate, create a new one:

    * Click Kubernetes Manifest and download the YAML file.
    
    * Apply the YAML in your *OpenShift cluster* using the *oc command*.

     ![image](https://github.com/user-attachments/assets/0b5089df-68af-4696-add0-5aac47a8fa73)
     
    * Verify the running Harness pod in the harness-delegate-ng namespace.

     ![image](https://github.com/user-attachments/assets/76e0e5cd-5c42-448b-b20e-1c3b93737c40)

    * Once complete, click Verify and Done.

* Select the delegate and click Save and Continue.

* Complete the Connection Test and click Finish.

* Confirm all connectors are ready.

### Task 4: Create a Pipeline

#### Go to Pipelines and click + New Pipeline.

* Name the pipeline: *WebApp-CI-CD*. Click Start.

#### Add Build Stage:

* Click + Add Stage and choose Build.

* Name the stage: *WebApp-Build.*

* Select Git Provider: *my-github-repo connector*

* Provide repository name: *dockerdemo* (Use your code repo name).

* Click Set Up Stage.

##### Configure Infrastructure

* Select Kubernetes Infrastructure.

* Choose Kubernetes Cluster Connector: *my-ocp-cluster*

* Set the namespace: *ci-cd-project* (Ensure the project/namespace exists in OpenShift).

* Click Continue.

##### Build and Push Docker Image

* Select Build and Push an Image to Docker Registry.

* Provide the name.

* Choose the Docker Connector: *my-docker-registry*

* Set the Docker Repository: *ocpctregistry/webapp*

* Set the tag: latest.

* Click Apply Changes.

#### Add Deploy Stage

* Click + Add Stage and choose Deploy.

* Name the stage: *WebApp-Deploy*

* Select Deployment Type: *Kubernetes*

* Click Set Up Stage.

##### Add Service

* Name the service: *Service-1*

* Specify Manifest Type: *Kubernetes*

* Choose Manifest Store: *GitHub*

* Select GitHub Connector: *my-github-repo*

* Click Continue.

* Set the manifest details:

    * Manifest Identifier: *webapp*
    
    * Repository Name: *git-repo*
    
    * Branch: *main*
    
    * File: *app.yaml*

* Click Submit. Click Save.

##### Configure Environment

* Click Continue and create a new environment.

* Provide the name and select the type.

* Click Save.

##### Specify Infrastructure

* Create new infrastructure.

* Provide the name and select Direct Connection Kubernetes.

* Choose Kubernetes Cluster Connector: *my-ocp-cluster*

* Set the namespace: *ci-cd-project*

* Click Save.

##### Select Execution Strategy

* select Execution Strategies as *Rolling* . Click Save.

### Task 4: Run the Pipeline

* Click Run Pipeline.

* Select the Git branch: *main*

* Click Run Pipeline.

* Monitor the execution using the Console View.

### Task 5: Verify Deployment

* Confirm the CI/CD pipeline ran successfully.

* Go to your OpenShift cluster.
![image](https://github.com/user-attachments/assets/976141b6-0f70-4b10-b102-1221a30db961)

 
* Verify the deployed application and access it through the defined routes.

* Copy the route URL and open it in a browser to view the application.
![image](https://github.com/user-attachments/assets/c3c55ef2-ffb1-4e5c-83bd-cccb2a647ac5)

