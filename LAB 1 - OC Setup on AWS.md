# Deploying an OpenShift Cluster on AWS using an EC2 instance

Prerequisites:
-------------------------------------------------------------------------------------
1. ***AWS Account:*** Ensure you have an AWS account.
2. ***Red Hat Free Subscription:*** Obtain a Red Hat Free Subscription.
3. ***IAM User with AdminAccess Permission:*** Create an IAM user with AdminAccess permission, and download its AccessKey and SecretKey.
4. ***Domain Name:*** Register a domain name from Bigrock, Hostinger or get a free one from Freenom.
-------------------------------------------------------------------------------------

## Task 1 : steps to create a Hosted zone in AWS Route 53
  * Log in to AWS Management Console.
    * Navigate to Route 53.
    * Click on "Hosted zones" in the left-hand navigation pane.
    * Enter the Domain Name (e.g., devopstraining.in.net).
    * Choose Public hosted zone.
    * Click "Create hosted zone".
    * Note the NS (Name Servers) records.
  * Update your domain's name servers to the ones provided in the Route 53 hosted zone at your domain registrar.
  * Verify DNS Propagation (https://www.whatsmydns.net/#NS/devopstraining.in.net)

## Task 2 : AWS EC2 Instance Configuration:-

1. ***OS:*** Use RHEL 9
2. ***Instance Type:*** t2.xlarge
3. ***Disk:*** 20 GB
4. Security Group Rules:
   * Allow HTTP (port 80)
   * Allow HTTPS (port 443)
   * Allow port 6443 (used for OpenShift).

## Task 3 :SSH into the EC2 Instance and run the following commands

Switch to Root User:
```
sudo su -
```
Set Hostname:
```
hostnamectl set-hostname openshift
```
```
bash
```
Update and Install Packages:
```
yum update -y
```
```
yum install wget awscli -y
```
*OR* Install AWS CLI via pip:

```
yum install python3-pip  -y
```
```
pip3 install awscli
```
Veryfy the Installation:
```
aws --version
```
```
aws configure
```
Generate SSH Key:
```
ssh-keygen -t ed25519 -N ''
```
starts the SSH agent in the background:
```
eval "$(ssh-agent -s)"
```
Adds the generated SSH key to the SSH agent.
```
ssh-add ~/.ssh/id_ed25519
```
## Task 4 : Obtain OpenShift Cluster Configuration 
 * Log in to https://console.redhat.com/openshift/ with your credentials.
 * Select "Cluster List" from the Navigation Pane and Click on "Create Cluster"
 * select "AWS(x86_64)" under "Run it Yourself."
 * Choose "Automated (CLI-based) installer-provisioned infrastructure."
 * Download Installer, Command Line Tool, and Pull Secret

## Task 5 : Download and Extract OpenShift Installer and CLI

```
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
```
```
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
```
Extract Files and Verify:
```
tar -xvf openshift-install-linux.tar.gz
```
```
tar -xvf openshift-client-linux.tar.gz
```
Move Extracted Binaries to /usr/local/bin:
```
mv openshift-install oc kubectl /usr/local/bin/
```

## Task 6 : Create OpenShift Cluster Configuration File:
```
openshift-install create install-config
```

Modify the configuration file if needed (i.e Number of replicas, machine type, and zones etc..)
```
vi install-config.yaml
```
refer below yaml:

```yaml
additionalTrustBundlePolicy: Proxyonly
apiVersion: v1
baseDomain: devopstraining.in.net
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform:
    aws:
      type: t3.xlarge
      zones:
      - us-east-2a
      ebs:
        volumeType: gp2      # Volume type (e.g., gp2, io1, etc.)
        volumeSize: 20       # Size of the volume in GB
        deleteOnTermination: false  # Volumes are not deleted on termination
  replicas: 3
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  platform:
    aws:
      type: t3.2xlarge
      zones:
      - us-east-2a
      ebs:
        volumeType: gp2      # Volume type (e.g., gp2, io1, etc.)
        volumeSize: 20       # Size of the volume in GB
        deleteOnTermination: false  # Volumes are not deleted on termination
  replicas: 1
metadata:
  creationTimestamp: null
  name: demo-cluster
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:
    region: us-east-2
publish: External
pullSecret: '{"auths":{"cloud.openshift.com":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfNjc0MzJlY2NiOTE3NDkwODhkZTMwZDQ1ZWRlNzc0M2U6QzQxRFRLMFhXQ0EyUTVEVkFET0gyME9QUEpDUlZIWEE3TlhOU0pVMVNYNTlOVkxNM1Q1WENMQ0o3VlRZTVkxVQ==","email":"cloudthatlinux03@gmail.com"},"quay.io":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfNjc0MzJlY2NiOTE3NDkwODhkZTMwZDQ1ZWRlNzc0M2U6QzQxRFRLMFhXQ0EyUTVEVkFET0gyME9QUEpDUlZIWEE3TlhOU0pVMVNYNTlOVkxNM1Q1WENMQ0o3VlRZTVkxVQ==","email":"cloudthatlinux03@gmail.com"},"registry.connect.redhat.com":{"auth":"fHVoYy1wb29sLTI1MTViZmVmLTEzZjQtNGYzNi1iZGM4LTAxMjdkNzE3OTU3ODpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXpNMkU1TldObE1HUTFPRGswTnpCaU9EY3pNV05oTkdKaU9HVmpZekptWlNKOS53WUg5QnlsQm5SbjRVTm1hajQ3WEZnWnZleE5NZWprc0VaSlhtSXJWcU5vSEhIUWRFUmNySXRnTVFyMFBJSWFCTjY4MmpjRm9EY09tYWYzQ082WmFiTVd5OW9Kc082ZHcyMjFVek5fNks0angwM2xVa3BaX1F3WEZFWDRTOUd3XzUzVXo0QzV3N1Rhc1RDRWdoMUlNUHRKUHRRYU5yYXR0T2hZa1duUDNpMFJwai1pYmFCWDZYVDgtZ1FiTGgxd3hqV28wZzM2UlFLcDk1R3dCWDl3ZEE2VjNoWjExYXB0Y083Y19nd2JfTDh3Wm53c1hJeTNXYmQzZ3hDQmdvbGMzZ3BfS2FJWXlzR1dBeDlaWW1zQmJnRWtuX1RIb196RkQtci1sVUdzZWNHamZNU0RCSTZLS2FGQ0pqcGppTnV4RmRTbmwwbkxLVm1rTUJtMThMMjNxSEFMdGlYdi11XzQyRWhEV2xFSGNqYV9sTHQ0cTVoamlQcHgwNmowY3FsWFNuY1RDdUlXTms4OS1UcHdHWU94SFgxa1hXb1Rub3VJN2xmcEcyR2dmY3d2dlVwZjk4dVBIUlRqS2ZlaEJVOVRUXzUtZmwxT2RKVlRMT2RlYnFOLTlCY2xIWjQ0RDNqRW5HVkRUSFYyRWZoSWVOaWVRNUxEbmZzYmp1RFNRaFJlaldoclZrMFFjbHBEWks2VkVlR2g3WjFzbjAzdUh5UWZvSnNmaDBPY1BRQmNOdFoxMzVZRDdubVhHOUd2Y21yMmEzc3dQd3Z1ZElPZHYxY0Z3Q0pRWUprSEJJcGxmMmppVzljWXJzZC1icWZDUnNYNFUtZ1EwSGxMQU5FX0pVY1JSZXd0VWQ5MExWLUpCbS1fcjR3ZXRwZVVFNGdzNTc0emp3ekVhcDRxQ2E3QQ==","email":"cloudthatlinux03@gmail.com"},"registry.redhat.io":{"auth":"fHVoYy1wb29sLTI1MTViZmVmLTEzZjQtNGYzNi1iZGM4LTAxMjdkNzE3OTU3ODpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXpNMkU1TldObE1HUTFPRGswTnpCaU9EY3pNV05oTkdKaU9HVmpZekptWlNKOS53WUg5QnlsQm5SbjRVTm1hajQ3WEZnWnZleE5NZWprc0VaSlhtSXJWcU5vSEhIUWRFUmNySXRnTVFyMFBJSWFCTjY4MmpjRm9EY09tYWYzQ082WmFiTVd5OW9Kc082ZHcyMjFVek5fNks0angwM2xVa3BaX1F3WEZFWDRTOUd3XzUzVXo0QzV3N1Rhc1RDRWdoMUlNUHRKUHRRYU5yYXR0T2hZa1duUDNpMFJwai1pYmFCWDZYVDgtZ1FiTGgxd3hqV28wZzM2UlFLcDk1R3dCWDl3ZEE2VjNoWjExYXB0Y083Y19nd2JfTDh3Wm53c1hJeTNXYmQzZ3hDQmdvbGMzZ3BfS2FJWXlzR1dBeDlaWW1zQmJnRWtuX1RIb196RkQtci1sVUdzZWNHamZNU0RCSTZLS2FGQ0pqcGppTnV4RmRTbmwwbkxLVm1rTUJtMThMMjNxSEFMdGlYdi11XzQyRWhEV2xFSGNqYV9sTHQ0cTVoamlQcHgwNmowY3FsWFNuY1RDdUlXTms4OS1UcHdHWU94SFgxa1hXb1Rub3VJN2xmcEcyR2dmY3d2dlVwZjk4dVBIUlRqS2ZlaEJVOVRUXzUtZmwxT2RKVlRMT2RlYnFOLTlCY2xIWjQ0RDNqRW5HVkRUSFYyRWZoSWVOaWVRNUxEbmZzYmp1RFNRaFJlaldoclZrMFFjbHBEWks2VkVlR2g3WjFzbjAzdUh5UWZvSnNmaDBPY1BRQmNOdFoxMzVZRDdubVhHOUd2Y21yMmEzc3dQd3Z1ZElPZHYxY0Z3Q0pRWUprSEJJcGxmMmppVzljWXJzZC1icWZDUnNYNFUtZ1EwSGxMQU5FX0pVY1JSZXd0VWQ5MExWLUpCbS1fcjR3ZXRwZVVFNGdzNTc0emp3ekVhcDRxQ2E3QQ==","email":"cloudthatlinux03@gmail.com"}}}'
sshKey: |
  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG9uXqckU6HnoZ0YhFktTzpAKGLnItNv00alecK0lI/z root@openshift

```
Create Installation Directory:
```
mkdir install-dir
```

Copy Configuration File to Installation Directory:
```
cp install-config.yaml install-dir/
```
Navigate to Installation Directory:
```
cd install-dir/
```
## Task 7 : Create OpenShift Cluster:

Run the following command to create `Openshift Cluster`
```
openshift-install create cluster --dir=/root/install-dir --log-level=debug
```
Export Cluster Credentials:
creates the `.kube` directory in the root user's home directory
```
mkdir /root/.kube
```
Copy the kubeconfig file, which contains the cluster credentials and configuration, into this directory
```
cp /root/install-dir/auth/kubeconfig /root/.kube/config
```
Alternatively, 
```
export KUBECONFIG=/root/install-dir/auth/kubeconfig
```

Run following commands to Verify Cluster Configuration:

```
oc get nodes
```
```
kubectl get nodes
```
```
oc new-app --image nginx
```
```
oc get pods
```

## Task 8 : Log in to the cluster by using the Web Console (GUI):

Run the following command to get Login Username and Password
```
cat .openshift_install.log | grep user
```
OR
```
 echo $(cat ~/install-dir/auth/kubeadmin-password)
```
Run the following command to get Web Console
```
oc get routes -n openshift-console  
```
Now, paste the URL into the browser, it will ask for username and password, enter them and you will get your console ready.

## Destroy the Cluster 

Run the following command to destroy `Openshift Cluster`
```
openshift-install destroy cluster --dir=/root/install-dir --log-level=debug
```
