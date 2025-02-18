## OpenShift Build and Deployment

### Task 1 : Create an OpenShift Project

Create a new project:

```
oc new-project buildConfig-lab
```
Verify it
```
oc project

```
### Task 2 : Create an ImageStream

```
oc create imagestream demoapp
```
Verify the ImageStream creation
```
oc get is
```
Describe the ImageStream
```
oc describe imagestream demoapp
```
### Task 3 : Create a BuildConfig
Create a BuildConfig YAML file
```
vi buildconfig.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: build-config
spec:
  source:
    type: Git
    git:
      uri: "https://github.com/sirinali07/dockerdmo.git"
  strategy:
    type: Docker
    dockerStrategy:
      dockerfilePath: "Dockerfile"
  output:
    to:
      kind: ImageStreamTag
      name: "demoapp:latest"
```
save the file using `ESCAPE + :wq!`
Apply the BuildConfig
```
oc apply -f buildconfig.yaml
```
Verify the BuildConfig
```
oc get buildconfig
```
Start the Build
```
oc start-build build-config --follow
```
Verify the taged image in the ImageStream
```
oc describe imagestream demoapp
```

### Task 4: Deploy the Built Image

Create a Deployment YAML file
```
vi deployment.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demoapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demoapp
  template:
    metadata:
      labels:
        app: demoapp
    spec:
      containers:
      - name: demoapp
        image: image-registry.openshift-image-registry.svc:5000/buildConfig-lab/demoapp:latest
        ports:
        - containerPort: 8080
```
save the file using `ESCAPE + :wq!`
Apply the Deployment
```
oc apply -f deployment.yaml
```
Verify the Deployment
```
oc get deployments
```

### Task 5 : Expose the Application with a Service and Route
Run the following command to expose the deployment via a service:
```bash
oc expose deployment demoapp-deployment --port=8080 --name=demoapp-service
```
Verify the service
```
oc get svc
```

Next, we will create an OpenShift Route to expose the Service externally.

```bash
oc expose service demoapp-service
```
Get the Route URL

```bash
 oc get routes 
```
You should see the route with an associated URL.

Access the Route:
* Open your browser and paste the URL
* The application should be accessible via this route.

### Task 6 : Cleanup Resources

Delete the project:
```
oc delete project buildConfig-lab
```


