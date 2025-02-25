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

### Task 2: Custom Security Context Constraints (SCC) in OpenShift

***Create a Custom Security Context Constraint (SCC)***

Create the Custom Security Context YAML File

```
vi custom-nonroot-scc.yaml
```

Add the given content, by pressing `INSERT`

```yaml
  apiVersion: security.openshift.io/v1
  kind: SecurityContextConstraints
  metadata:
    name: custom-nonroot-scc
  allowPrivilegedContainer: false
  runAsUser:
    type: MustRunAsRange
    uidRangeMin: 2000
    uidRangeMax: 2010
  fsGroup:
    type: MustRunAs
    ranges:
      - min: 2000
        max: 2010
  supplementalGroups:
    type: MustRunAs
    ranges:
      - min: 2000
        max: 2010
  seLinuxContext:
    type: MustRunAs
  readOnlyRootFilesystem: false
  volumes:
    - emptyDir
    - hostPath
  allowHostDirVolumePlugin: true
```
save the file using `ESCAPE + :wq!`

Apply the yaml
```
oc apply -f custom-nonroot-scc.yaml
```
verify it
```
oc get scc | grep custom-nonroot-scc
```
***Create a Service Account***

Create a service account named `custom-scc-sa`
```
oc create sa custom-scc-sa
```

***Associate the service account***

Associate the service account with the custom SCC
```
oc adm policy add-scc-to-user custom-nonroot-scc -z custom-scc-sa
```
***Deploy a Pod Using the Custom SCC***

Create a pod definition file named `custom-scc-pod.yaml`

```
vi custom-scc-pod.yaml
```

Add the given content, by pressing `INSERT`

```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: custom-scc-test-pod
  spec:
    serviceAccountName: custom-scc-sa
    containers:
      - name: httpd
        image: registry.redhat.io/ubi8/httpd-24
        command: ["sleep", "3600"]
        securityContext:
          runAsUser: 2001
          runAsGroup: 2001
        volumeMounts:
          - name: my-emptydir
            mountPath: /data
          - name: my-hostpath
            mountPath: /app
    volumes:
      - name: my-emptydir
        emptyDir: {}
      - name: my-hostpath
        hostPath:
          path: /tmp/volume
```
save the file using `ESCAPE + :wq!`

Deploy the pod
```
oc apply -f custom-scc-test-pod.yaml
```
Confirm the pod is running
```
oc get pods
```
***Validate the SCC Assignment***

Describe the pod to verify the SCC
```
oc describe pod custom-scc-pod | grep scc
```
You should see custom-nonroot-scc assigned.

Access the running pod
```
oc rsh custom-scc-pod
```
heck the user ID and group ID
```
id
```
(Optional) Create a file in the mounted volume to confirm write access:
```
touch /data/testfile.txt
```
```
ls -l /data
```
Exit the pod
```
exit
```
### Task 3: Cleanup the resources

Delete the project:
```bash
oc delete project test-scc
```






