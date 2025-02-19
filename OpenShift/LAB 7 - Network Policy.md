
## Ingress Network Policy
### Task 1: Deploy HTTPD Application in Network Policy Namespace

Create a New Project
```
oc new-project network-policy
```
Deploy an HTTPD App
```
oc new-app registry.redhat.io/ubi8/httpd-24 --name=httpd-app --labels app=httpd
```
Lists all resources in the namespace to ensure the application is running
```
oc get all
```
Get Service Details
```
oc get svc
```
Note the service IP Address and Protocol.
Tests if the HTTPD service is reachable internally.
```
oc rsh httpd-app-<pod_id> curl <httpd-svc-ip>:8080
```
*i.e* oc rsh httpd-app-9d74c4945-wrzlp curl 172.30.214.47:8080

### Task 2: Deploy Another App in a Different Namespace

Creates another namespace for testing cross-namespace communication.
```
oc new-project external-namespace
```
Labels the namespace for use in network policies.
```
oc label ns external-namespace network=external
```
Deploys another HTTPD app in the different namespace.
```
oc new-app registry.redhat.io/ubi8/httpd-24 --name=webapp --labels app=webapp
```
Ensures the application has been deployed successfully.
```
oc get all
```
Checks if the webapp can reach the HTTPD service in `network-policy` namespace.
```
oc rsh webapp-<pod_id> curl <httpd-svc-ip>:8080
```
*i.e* oc rsh webapp-c4864b584-grpx2 curl 172.30.214.47:8080

### Task 3: Apply a Deny-All Network Policy
Switch to the `network-policy` namespace where the network policy will be enforced
```
oc project network-policy
```
Create a Deny-All Policy

```
vi deny-all.yaml
```
Add the given content, by pressing `INSERT`
```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all
spec:
  podSelector: {}
```
save the file using `ESCAPE + :wq!`

Defines a policy that blocks all ingress traffic to any pod in the namespace.

Apply the Deny-All Policy
```
oc create -f deny-all.yaml
```
Enforces the deny-all policy in the `network-policy` namespace.

Confirms that the policy has been successfully applied.
```
oc get networkpolicy
```
```
oc describe networkpolicy deny-all
```

Test Connectivity (Should Fail)
```
oc project external-namespace
```
```
oc rsh webapp-<pod_id> curl <httpd-svc-ip>:8080
```
*i.e* oc rsh webapp-c4864b584-grpx2 curl 172.30.214.47:8080
Attempts to access the HTTPD service from the `external-namespace` (should be blocked).

### Task 4: Create an Allow Policy for Specific Namespace
Switch to the Network Policy Namespace
```
oc project network-policy
```

Create an Allow Policy
```
vi allow-ns.yaml
```
Add the given content, by pressing `INSERT`
```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-specific-namespace
spec:
  podSelector:
    matchLabels:
      app: httpd
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          network: external
      podSelector:
        matchLabels:
          app: webapp
    ports:
    - port: 8080
      protocol: TCP
```
save the file using `ESCAPE + :wq!`

Applies the policy to allow traffic from external-namespace to network-policy-app
```
oc create -f allow-ns.yaml
```
Verify the Policy

```
oc get networkpolicy
```
```
oc describe networkpolicy allow-specific-namespace
```
Verifies the policy has been applied successfully.
Test Connectivity (Should Work for WebApp)
```
oc project external-namespace
```
```
oc rsh webapp-<pod_id> curl <httpd-svc-ip>:8080
```
*i.e* oc rsh webapp-c4864b584-grpx2 curl 172.30.214.47:8080

Should work, as webapp is allowed to access the HTTPD service in network-policy-app.
Should fail, as the other pod is not permitted to access the HTTPD service.

## Egress Network Policy

### Task 5: Create a Pod and Test Default Egress Behavior
Create a new pod 'nginx-pod' running the nginx image.
```
oc run nginx-pod --image=nginx:latest
```
Access the pod's terminal.
```
oc exec -it nginx-pod -- bash
```
Test if the pod can access external addresses by attempting to curl two different services:
```
curl https://8.8.8.8
```
```
curl https://yahoo.com
```

Exit the pod after testing.
```
exit
```

### Task 6: Create an Egress Network Policy
Create a new YAML file to define the egress rules. 

```
vi egress-policy.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-outbound
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 8.8.8.8/32  # Example IP address
```
save the file using `ESCAPE + :wq!`

The above policy allows outbound traffic only to the IP address 8.8.8.8 (Google DNS), and blocks all other egress traffic.

Apply the policy.

```
oc apply -f egress-policy.yaml
```
Verify Egress Network Policy


List all the network policies in the current namespace to verify that the egress policy is applied.
```
oc get networkpolicies
```
Describe the Egress Policy:
```
oc describe networkpolicies allow-outbound
```
Access the terminal of nginx-pod to check if the egress policy is applied correctly.
```
oc exec -it nginx-pod -- bash
```
Test the Allowed Egress Connectivity:
```
curl https://8.8.8.8
```
Test the Blocked Egress Connectivity:
```
curl https://yahoo.com
```
After applying the Egress Policy, the pod can only access 8.8.8.8, and all other outbound traffic is blocked.

Once the tests are complete, exit the pod.
```
exit
```
### Task 7: Cleanup Resources
 
Delete the project which will remove all resources within the project, so proceed with caution
```
oc delete project network-policy
```

```
oc delete project external-namespace
```




