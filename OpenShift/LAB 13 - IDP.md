
### Creating Users and Logging into OpenShift Using HTPasswd

### Task 1 : Install htpasswd

htpasswd is part of the httpd-tools package. Install it using the following command:

On RHEL/CentOS:
```
yum update && yum install -y httpd-tools
```
On Ubuntu/Debian:
```
apt-get update && apt-get install -y apache2-utils
```
Verify the installation
```
htpasswd -h
```

### Task 2 : Create the HTPasswd File

HTPasswd is used for managing users and passwords.

Create an htpasswd file with users (admin and devloper)
```
htpasswd -c -B -b htpasswd admin admin
```
```
htpasswd -b htpasswd devloper devloper
```
* -c : creates a new file.
* -B : uses bcrypt hashing.
* -b : allows inline passwords.

Verify the file
```
cat htpasswdfile
```

### Task 3 : Users in OpenShift

```bash
oc create user admin
```
```bash
oc create user devloper
```
Check if the users exist in OpenShift
```bash
oc get users
```
Describe a user to check their identity mapping
```
oc describe user admin
```
```
oc describe user devloper
```
### Task 4 : Create a Secret in OpenShift
Now, store the htpasswd file as a secret in OpenShift.
```
oc create secret generic myhtpasswdidp-secret --from-file=/root/htpasswd -n openshift-config
```
*Note : Make sure the path to the `htpasswd file` is correctly specified.*

Verify the secret
```
oc get secret myhtpasswdidp-secret -n openshift-config
```

### Task 5 : Configure OAuth for HTPasswd Authentication
Now, edit the OpenShift authentication configuration.

Create a file named oauth.yaml
```
oc get oauth -o yaml > oauth.yaml
```
```
vi oauth.yaml
```
edited `spec field` 

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: htpassidp  # Name of the Identity Provider (IDP)
    type: HTPasswd  # Defines the authentication type as HTPasswd
    htpasswd:
      fileData:
        name: myhtpasswdidp-secret  # Name of the secret storing the htpasswd file
```
The structure remains the same, only modifying the `spec` section.

Apply the updated OAuth configuration
```
oc apply -f oauth.yaml
```
Restart the authentication pods to apply changes:
```
oc delete pod -n openshift-authentication --all
```
Verify the updates
```
oc get pod -n openshift-authentication 
```


### Task 6 : Assign Roles to Users
By default, new users have no permissions.
*Assign `cluster-admin` Role to `admin`*
```
oc adm policy add-cluster-role-to-user cluster-admin admin
```
Check assigned roles
```
oc get clusterrolebindings | grep admin
```
***Create a Custom Role for `Developer`***
```
oc create role dev-role --verb=view,delete,create --resource=pods,services
```
*Assign the `dev-role` to the `developer` user*
```
oc adm policy  add-role-to-user dev-role devloper
```
Check assigned roles
```
oc get rolebinding -n app-project 
```

### Task 7 : Verify User Login
**Test logging in with the admin user**
```
oc login -u admin -p admin
```
Check current user:
```
oc whoami
```
```
oc get nodes
```
This should work since cluster-admin has full access
```
oc logout
```
**For devloper**
```
oc login -u devloper -p devloper
```
Check current user:
```
oc whoami
```
```
oc get po
```
```
oc logout
```

### Task 8 : Clean Up
If you want to remove the users:
```
oc delete user admin devloper
```
```
oc delete identity htpassidp:admin htpassidp:devloper
```
