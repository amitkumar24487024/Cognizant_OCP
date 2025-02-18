### Managing and Deploying Applications with ImageStreams in OpenShift

### Task 1 : Create an OpenShift Project

Create a new project:

```
oc new-project image-trigger-lab
```
Verify it
```
oc project

```
### Task 2 : Create an ImageStream

```
oc create imagestream myapp
```
Verify the ImageStream creation
```
oc get is
```
Tag an image from `Docker Hub` to the `ImageStream`
```
oc tag docker.io/sirin07ali/my-nginx:v1 myapp:latest
```
Verify the ImageStream
```
oc describe imagestream myapp
```

### Task 3 : Deploying with `DeploymentConfig`
Create a DeploymentConfig YAML File
```
vi deploymentconfig.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata:
  name: nginx-dc
  labels:
    app: nginx-dc
spec:
  replicas: 2
  selector:
    app: nginx-dc
  template:
    metadata:
      labels:
        app: nginx-dc
    spec:
      containers:
      - name: nginx-container
        image: myapp:latest  # References the ImageStreamTag
        ports:
        - containerPort: 80
  triggers:
  - type: ImageChange  # Defines an ImageChange trigger for this DeploymentConfig
    imageChangeParams:
      automatic: true  # Enables automatic redeployment when the ImageStreamTag is updated
      containerNames:
      - nginx-container  # Specifies the container(s) in the DeploymentConfig that should use the updated image
      from:
        kind: ImageStreamTag  # Indicates the type of resource being referenced (ImageStreamTag in this case)
        name: myapp:latest  # Specifies the tag in the ImageStream to watch for updates

```
save the file using `ESCAPE + :wq!`
Apply the DeploymentConfig
```
oc apply -f deploymentconfig.yaml
```
Verify the DeploymentConfig
```
oc get dc
```
```
oc get pods 
```
Expose the Deployment:
```
oc expose dc nginx-dc --port=80
```
```
oc get svc
```
```
oc expose svc nginx-dc 
```
```
oc get route
```
Access the Application:

Copy the route URL displayed in the output and open it in a browser to view the application.

### Task 4 : Update the ImageStream
Tag a new image   to myapp:latest to simulate a new release
```
oc tag docker.io/sirin07ali/my-nginx:v2 myapp:latest
```
Verify the New Image in ImageStream
```
oc describe imagestream myapp
```
### Task 5 : Watch the Automatic Redeployment
Monitor the pods and observe a rolling update:
```
oc get pods -w
```
You should see the old pods being replaced with new ones.

Describe one of the newly created pods to confirm it uses the updated image
```
oc describe pod <pod-name>
```
Look for the `Image` field under the container details
OR
Access the application:
Copy the route URL from the output of `oc get route` and open it in a browser to ensure the application is functioning with the updated image.

### Task 6 :  Cleanup
Remove all resources created during the lab:
```
oc delete all -l app=nginx-dc
```
Delete the project:
```
oc delete project image-trigger-lab
```

-----------------------------------------------------------------------------
### When to Use Which?
**Deployment:**
* If you plan to maintain Kubernetes compatibility or are using advanced Kubernetes features.
* Preferred for simpler and more Kubernetes-native deployment workflows.
  
**DeploymentConfig:**
* Use when you need advanced triggers (e.g., automatic build/image updates).
* If you are integrating with older OpenShift workflows or need custom deployment strategies.
  
**Key Differences**
![image](https://github.com/user-attachments/assets/f4f1adb8-cb7d-4551-b7d8-ddfe9c59cb0d)

-----------------------------------------------------------------------------
### ImageStream in OpenShift

An ***ImageStream*** is a way to manage and track container images in OpenShift. It provides an abstraction layer over container images, allowing for easier management, versioning, and integration with OpenShift deployments.

**ImageStreamTags:** Images within an ImageStream are identified by tags (e.g., v1, latest). Each tag points to a specific image version, either local or remote (e.g., from Docker Hub).

**Automatic Updates:** ImageStreams can automatically trigger deployments when an image is updated (via an ImageChange trigger in DeploymentConfig).
