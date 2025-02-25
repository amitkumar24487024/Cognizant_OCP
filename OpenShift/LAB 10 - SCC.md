## Security Context Constraints (SCCs) in OpenShift

### Task 1:  SCCs in OpenShift
Create a new project called `test-scc`
```
oc new-project test-scc
```
Create an Nginx deployment  
```
oc create deployment ng-dep --image=nginx
```
Get and Describe the Nginx Pod
```
oc get po
```
```
oc describe pod ng-dep-dc7c5ff85-nr5rp
```
You can observe that the pod will not run because by default Docker Hub images are configured to run as a non-root user, which is blocked by OpenShift’s default SCC settings
To allow the pod to run successfully, the anyuid SCC needs to be applied to the service account.

List all the existing SCCs available in your OpenShift cluster.
```
oc get scc
```
Get detailed information about the anyuid SCC.
```
oc describe scc anyuid
```
***Create a Service Account (SA)***

```
oc create sa nginx-sa
```
***Associate the service account***

Apply the anyuid SCC to the Service Account
```
oc adm policy add-scc-to-user anyuid -z nginx-sa
```
This allows pods using the service account to run with the anyuid SCC, enabling them to run as any UID, including root, for containers requiring elevated privileges.

***Assign the Service Account to the Deployment***

```
oc set serviceaccount deployment/ng-dep nginx-sa
```
List the pods in the project
```
oc get po
```
After applying the service account to the deployment, list the pods to check if the pod now uses the nginx-sa service account and running

### Task 2: Cleanup the resources

Delete the project:
```bash
oc delete project test-scc
```






