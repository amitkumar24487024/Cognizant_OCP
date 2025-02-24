
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

    
