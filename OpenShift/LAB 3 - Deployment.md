
### Task 1 : Create an OpenShift Project

Create a new project:

```
oc new-project deployment-project
```
Verify it
```
oc project

```
### Task 2 : Deploying with `Deployment`
Create a Deployment YAML File
```
vi deployment.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-ctr
        image: nginx:1.24.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "400Mi"
            cpu: "350m"
```
save the file using `ESCAPE + :wq!`

Apply the Deployment
```
oc apply -f deployment.yaml
```
Verify the Deployment:
```
oc get deployment
```
```
oc get pods -l app=nginx
```

### Task 3 :  Cleanup
Remove all resources created during the lab:
```
oc delete all -l app=nginx
```

Delete the project:
```
oc delete project deployment-project
```
