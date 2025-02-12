### Kubernetes CLI (kubectl):

**Kubectl** is the command-line tool used to interact with Kubernetes clusters.
* Manage Kubernetes resources like Pods, Services, Deployments, ConfigMaps, etc.
* Supports YAML/JSON input for resource definitions.

**Basic Commands:**

* Get cluster information
```
kubectl cluster-info
```
* List all pods in a namespace
```
kubectl get pods -n <namespace>
```

* Apply a YAML file to create resources
```
kubectl apply -f <file.yaml>
```

* Describe a pod for debugging
```
kubectl describe pod <pod-name> -n <namespace>
```

* Delete a resource
```
kubectl delete <resource-type> <resource-name> -n <namespace>
```
### OpenShift CLI (oc):

**oc** is an enhanced CLI built on top of kubectl, tailored for OpenShift.
* Manages OpenShift-specific resources like Routes, Projects, and Builds.
* Integrates seamlessly with OpenShift's Source-to-Image (S2I) feature.
* Provides authentication for OAuth-based OpenShift clusters.

**Basic Commands:**
* Log in to an OpenShift cluster
```
oc login <cluster-url> --token=<token>
```

* List all projects
```
oc get projects
```

* Switch to a project
```
oc project <project-name>
```

* Get all resources in a project
```
oc get all -n <namespace>
```
* Create a new project
```
oc new-project <project-name>
```

* Create a route to expose a service
```
oc expose svc <service-name>
```

* Build and deploy an app using S2I
```
oc new-app <image-stream>/<app-name>:<tag> --name=<app-name>
```

### Task 1 : Create and Explore a Pod Using `kubectl` Command
Use the following command to create a Pod named `pod-1` running the `nginx` container image:
```
kubectl run pod-1 --image nginx --port 80 
```
Check check the status of newly created Pod
```
kubectl get pod
```
Use the -o wide flag to get additional details
```
kubectl get pod -o wide
```
View detailed information about the Pod
``` 
kubectl describe pod pod-1
```

### Task 2 : Create and Explore a Pod Using `oc` Command
Use the following command to create a Pod named `pod-1` running the `nginx` container image:
```
oc run oc-pod --image=nginx --port=80
```
Check check the status of newly created Pod
```
oc get pod
```
Use the -o wide flag to get additional details
```
oc get pod -o wide
```
View detailed information about the Pod
``` 
oc describe pod oc-pod
```
Check the logs of the containers
``` 
oc logs oc-pod
```
Enter the pod to explore it further
``` 
oc exec oc-pod -it -- /bin/bash
```
```
exit
```
Expose the pod to be access externally
``` 
oc expose pod oc-pod --type=LoadBalancer --port=80 --name oc-svc
```
```
oc get svc
```
Copy paste the LoadBalancer URL in the browser to access pod application
