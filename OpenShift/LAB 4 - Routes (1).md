## OpenShift Route Configuration
In OpenShift, Routes are used to expose services externally, especially for HTTP-based traffic.

### Task 1 : Create an OpenShift Project

Create a new project:

```
oc new-project route-lab
```
Verify it
```
oc project
```
You should see `route-lab` as the current project.

### Task 2 : Create a Pod
Create a new file named httpd-pod.yaml
```
vi httpd-pod.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd
    environment: production
spec:
  containers:
  - name: httpd-container
    image: httpd
    ports:
      - containerPort: 80
```
save the file using `ESCAPE + :wq!`

Apply the Pod configuration

```bash
 kubectl apply -f httpd-pod.yaml
```
Verify the Pod creation

```bash
kubectl get pods
```
You should see httpd-pod running.

Describe the Pod to check details
```bash
 kubectl describe pod httpd-pod
```
### Task 3 : Create a service
Now, we expose the Pod using a Service. The service allows internal communication with the Pod.

```bash
kubectl expose pod httpd-pod --port 80 --name httpd-svc
```
Verify the service
```
kubectl get svc
```

Next, we will create an OpenShift Route to expose the Service externally.

### Task 4 :  OpenShift Route Configuration
Create a Route YAML File named `httpd-route.yaml`

```bash
vi httpd-route.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: httpd-route
spec:
  to:
    kind: Service
    name: httpd-svc     # Internal service to route traffic to
  port:
    targetPort: 80      # Target port on the service

```
save the file using `ESCAPE + :wq!`

Apply the Route configuration

```bash
oc apply -f httpd-route.yaml
```
`Alternatively`, you can create the `route` using the following command

```
oc expose service httpd-svc --name httpd-route
```

Verify the Route

```bash
 oc get routes 
```
You should see the route with an associated URL.

Access the Route:
* Open your browser and navigate to the provided URL for the Route. 
* You should see the default Apache web page served by the Pod.


### Task 5 : Cleanup Resources
Delete the Route (in OpenShift):
```bash
oc delete -f httpd-route.yaml
```
Delete the Service (in Kubernetes):
```bash
kubectl delete svc httpd-svc
```
Delete the Pod:
```bash
kubectl delete -f httpd-pod.yaml
```
Delete the project:
```
oc delete project route-lab
```




