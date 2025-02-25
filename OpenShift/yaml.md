
## hostPath
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    app: webapp
spec:
  containers:
  - name: app
    image: 'nginx:latest'
    volumeMounts:
    - mountPath: '/usr/share/nginx/html/'
      name: hostpath-volume
  volumes:
  - name: hostpath-volume
    #volume type:
    hostPath:
      path: /volume #existing dir
```

## emptyDir
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-ctr-app
  labels:
    app: webapp
spec:
  containers:
  - name: nginx
    image: 'nginx:latest'
    volumeMounts:
    - mountPath: '/usr/share/nginx/html/'
      name: emptydir-volume
  - name: busybox
    image: 'busybox'
    command: ['sh', '-c', 'sleep 6000']
    volumeMounts:
    - mountPath: '/app'
      name: emptydir-volume
  volumes:
  - name: emptydir-volume
    #volume type:
    emptyDir: {}
```
## PV, PVC, and POD
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: test-sc
  hostPath:
    path: /volume
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: test-sc
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  volumes:
  - name: test-vol
    persistentVolumeClaim:
      claimName: test-pvc
  containers:
  - name: pv-recycler
    image: "registry.k8s.io/busybox"
    command: ["/bin/sh", "-c", "test -e /scrub && rm -rf /scrub/..?* /scrub/.[!.]* /scrub/*  && test -z \"$(ls -A /scrub)\" || exit 1"]
    volumeMounts:
    - name: test-vol
      mountPath: /scrub
```
### Multi-ctr with Security Context
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-ctr-app
  labels:
    app: webapp
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 5000
  containers:
  - name: app
    image: httpd:latest
    volumeMounts:
    - mountPath: '/app'
      name: emptydir-volume
  - name: data
    image: httpd:latest
    volumeMounts:
    - mountPath: '/dat'
      name: emptydir-volume
  volumes:
  - name: emptydir-volume
    emptyDir: {}
```
    
