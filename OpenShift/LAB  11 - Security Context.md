## Security Context

## TAsk 1 : Pod & Container Level Security Context
```
vi security-context.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context
spec:
  securityContext:
    fsGroup: 2000     # common group of files
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: ctr-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      runAsUser: 1010   #UID
      runAsGroup: 3010  #GID
  - name: ctr-2
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /docs/demo
    securityContext:
      runAsUser: 1020   #UID
      runAsGroup: 3020  #GID
```
save the file using `ESCAPE + :wq!`

Create the Pod:
```
kubectl apply -f security-context.yaml
```
Verify that the Pod's Containers are running:
```
kubectl get po
```
Get a shell into the ctr-1 Container:
```
kubectl exec -it security-context -c ctr-1 -- sh
```
```
id
```
```
cd /data/demo/
```
```
echo hello > file1.txt
```
```
ls -l
```
```
exit
```
Get a shell into the ctr-2 Container:
```
kubectl exec -it security-context-new -c ctr-2 -- sh
```
```
id
```
```
cd /docs/demo/
```
```
echo hello > file2.txt
```
```
ls -l
```
```
exit
```
## TAsk 2 : Cleanup Resource
```
kubectl delete -f security-context.yaml
```
