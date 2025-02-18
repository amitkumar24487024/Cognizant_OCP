## Build application with Source-to-Image(S2I)

### Task 1 : Create an OpenShift Project

Create a new project:

```
oc new-project s2i-project
```
Verify it
```
oc project

```

### Task 2 : Deploy the Application Using S2I

Check your Git repository for the code.

![image](https://github.com/user-attachments/assets/4f48e2d3-fa29-4ad9-be9e-43fe5a7b6dbf)

Use the S2I process to create an application based on your Git repository:

```
oc new-app https://github.com/sirinali07/Hello-World-Repo.git  --name hello-world  -l app=NodeJS-app
```
Make sure to replace the Git repository URL with your own. If you don't have a repository yet, create one and upload the code provided in the repo https://github.com/sirinali07/Hello-World-Repo.git

Retrieve all resources (pods, services, deployments, etc.) associated with the label app=NodeJS-app

```
oc get all -l app=NodeJS-app
```
Alternatively, 
Check the status of the build and Pods:

```
oc get build
```
```
oc get pod
```

Check the Services:

```
oc get svc
```

Check the Route:

```
oc get route
```

You can see that no route has been created yet.
Create a route to expose the application:

```
oc expose service  nodejs-app
```

Get the application URL:

```
oc get route
```

Open a web browser and access the application at the URL provided in the previous step (e.g., hello-world-s2i-project.apps.demo-cluster.devopstraining.in.net).

![image](https://github.com/user-attachments/assets/b602d9a3-02f9-4971-a81e-22e146a55140)


### Task 3 : Clean Up

Delete the application and its associated resources:
```
oc delete all -l app=NodeJS-app
```
Delete the project:
```
oc delete project s2i-project
```
