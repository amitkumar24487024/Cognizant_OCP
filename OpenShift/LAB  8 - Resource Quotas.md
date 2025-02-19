## Resource Quotas

### Task 1 : Create an OpenShift Project

Create a new project:

```
oc new-project quota-lab
```
Verify it
```
oc project

```

### Task 2: Creating Resource Quota and Constraining Object Creation

Create a pod and expose it, before applying the resource quota to check if the resource quota applies to existing objects or not.
```
oc  run pod1 --image nginx --port 80
```
```
oc expose pod pod1 --name pod1-svc --port 80 --type NodePort 
```


**Imperative**
```
oc create quota rs-quota1 --hard=pods=2,services=1
```
```
oc describe ns quota-lab
```
```
oc get quota -n quota-lab
```
**Declarative** (OR)
```
vi rq1.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota1
  namespace: quota-lab
spec:
  hard:
    pods: "2"
    services: "1"

```
```
oc apply -f rq1.yaml
```
**Note- Either create Quota with **Imperative Way** OR **Declarative Way**
```
oc describe ns quota-lab
```
```
oc get quota -n quota-lab
```
```
oc describe quota -n quota-lab
```
```
oc -n quota-lab run pod2 --image nginx --port 80
```
```
oc -n quota-lab expose pod pod2 --name pod2-svc --port 80 --type NodePort 
```
Try to deploying a new pod in namespace quota-lab
```
oc -n quota-lab run pod3 --image nginx --port 80
```
**Once the the desire quota acheaved, It will not allow you to exceed the limit and you will get Frobidden Message.
##### Delete the quota, svc & pods created in previous steps
```
oc delete quota rs-quota1 -n quota-lab
```
```
oc -n quota-lab delete pod --all
```
```
oc -n quota-lab delete svc --all
```

### Task 3: Creating Resource Quota and Constraining Hardware Resources

```
vi rq2.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota2
  namespace: quota-lab
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    pods: "2"
    services: "1"
    count/deployments.apps: "1"
```
```
oc apply -f rq2.yaml
```
```
oc describe ns quota-lab
```
```
oc describe quota -n quota-lab
```
### Task 4: Verify Resource Quota Functionality
```
oc -n quota-lab run pod4 --image nginx --port 80
```
```
vi pod5.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod5
  namespace: quota-lab
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      requests:
        memory: "600Mi"
        cpu: "350m"
    ports:
      - containerPort: 80
```
```	  
oc apply -f pod5.yaml
```
```
oc describe ns quota-lab
```
```
oc describe quota -n quota-lab
```
### Task 5 : Cleanup Resources

Delete the project:
```
oc delete project quota-lab
```

