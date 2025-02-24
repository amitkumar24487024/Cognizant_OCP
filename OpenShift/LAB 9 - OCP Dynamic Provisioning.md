 ## Dynamic Provisioning, Snapshot, and Disaster Recovery with AWS EBS (gp3)

### Task 1 : Create an OpenShift Project

Create a new project:

```
oc new-project dynamic-provisioning-lab
```
Verify the current project
```
oc project
```
You should see `dynamic-provisioning-lab` as the current project.

### Task 2: Create a Storage Class 
StorageClasses allow dynamic provisioning of AWS EBS volumes.

Create the Storage Class YAML File

```
vi storageclass.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-storage
provisioner: ebs.csi.aws.com   #AWS EBS CSI driver
parameters:
  type: gp3
  fsType: ext4
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```
save the file using `ESCAPE + :wq!`

Apply the storageclass definition yaml
```
oc apply -f storageclass.yaml
```
Verify the Storage Class
```bash
oc get sc
```

### Task 3: Create a PersistentVolumeClaim (PVC) using below yaml
A PVC allows users to request storage dynamically from the StorageClass.

Create the PVC YAML File
```
vi pvc.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ebs-storage
```
save the file using `ESCAPE + :wq!`

Apply the definition yaml
```
oc apply -f ebs-pvc.yaml
```

Verify the PVC
```bash
oc get pvc
```

### Task 4: Deploy a Pod Using the PVC
Runs an Apache `HTTPD container` and mounts the `EBS volume`

Create the Pod YAML File
```
vi ebs-pod.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ebs-dynamic-app
  labels:
    app: webapp
spec:
  containers:
  - name: app
    image: 'httpd:latest'
    volumeMounts:
    - mountPath: '/usr/local/apache2/htdocs/'
      name: ebs-volume
  volumes:
  - name: ebs-volume
    persistentVolumeClaim:
      claimName: ebs-dynamic-pvc
```
save the file using `ESCAPE + :wq!`

Apply the definition yaml
```
oc apply -f pod.yaml
```

Verify the POD
```bash
oc get po
```
Ensure the pod is in a `running` state.

### Task 5: Write Data to the Persistent Volume

Access the Pod
```bash
oc exec -it ebs-dynamic-app -- bash
```

Navigate to the Mounted Directory
```bash
cd /usr/local/apache2/htdocs/
```
Create an HTML file with sample content
```bash
echo "This is a test page deployed on an Apache server with persistent storage." > index.html
```

Exit the Pod
```bash
exit
```
### Task 6: Expose the Pod and Access the Web Page

Expose the pod internally as a service:
```bash
oc expose pod ebs-dynamic-app --port 80
```
Create route to expose the Service externally.
```bash
oc expose svc ebs-dynamic-app
```
Retrieve the route URL
```
oc get route
```
You should see the route with an associated URL.

***Access the Route:***
Open the URL in your browser. You should see the web page displaying the content from `index.html`.

### Task 7: Create a Snapshot Class
Create the Snapshot Class YAML File
```
vi gp3-snapshotclass.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: gp3-snapshot-class
driver: ebs.csi.aws.com
deletionPolicy: Retain
```
save the file using `ESCAPE + :wq!`

Apply it
```
oc apply -f gp3-snapshotclass.yaml
```
Verify the snapshot class
```
oc get volumesnapshotclass
```

### Task 8:  Create a Volume Snapshot
Let's take a snapshot of the PVC to back up the stored data.

Create the Snapshot YAML File

```
vi gp3-snapshot.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: gp3-snapshot
spec:
  volumeSnapshotClassName: gp3-snapshot-class
  source:
    persistentVolumeClaimName: ebs-dynamic-pvc
```
save the file using `ESCAPE + :wq!`

Apply it
```
oc apply -f gp3-snapshot.yaml
```
Verify the snapshot
```
oc get volumesnapshot
```

### Task 9: Restore the Volume from Snapshot

Restore the snapshot to a new PVC.

Create the Restored PVC YAML File
```
vi restore-pvc.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-pvc
spec:
  dataSource:
    name: gp3-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ebs-storage
```
save the file using `ESCAPE + :wq!`

Apply it
```
oc apply -f restore-pvc.yaml
```

Verify the restored PVC:
```bash
oc get pvc
```
### Task 10: Deploy a Pod to Verify Restored Data
Create a new pod and mount the restored volume.

```
vi restore-pod.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restore-test-pod
  labels:
    app: restore-testapp
spec:
  containers:
    - name: app
      image: 'httpd:latest'
      resources: {}
      volumeMounts:
        - name: ebs-volume
          mountPath: /usr/local/apache2/htdocs/
  volumes:
   - name: ebs-volume
     persistentVolumeClaim:
      claimName: restored-pvc
```
save the file using `ESCAPE + :wq!`

Apply it
```
oc apply -f restore-pod.yaml
```

Verify the POD Status
```bash
oc get pod
```

Check if the restored pod has the same data.

```bash
oc exec -it restore-test-pod -- cat /usr/local/apache2/htdocs/index.html
```
Expected Output:
***This is a test page deployed on an Apache server with persistent storage.***


### Task 11: Cleanup the resources


Delete the project:
```bash
oc delete project dynamic-provisioning-lab
```
