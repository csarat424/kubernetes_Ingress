# kubernetes_Ingress
This repo includes all the commands on how to secure communication between Users and Services with Ingress
Kops_Ingress_Project

Step 1 ---> Create a Kops Kubernetes Cluster

## Kops_kubernetes
This repo has a detailed step-by-step approach on how to initiate a production grade KOPS kubernetes cluster

1. DNS NAME - mscgov.xyz
2. S3 BUCKET - mscgov.xyz
3. CREATE A EC2 WITH T2.MEDIUM - Management Server
4. IAM ROLE AND ASSIGN IT TO EC2  -  Kops_Cluster
5. CONNECT TO  EC2 INSTANCE AND GENERATE ssh-keygen  
6. download Kops and Kubectl to usr/local/bin and change permission 
7. edit .bashrc and add all the below mentioned env variables

Kops link for linux amd 
```
https://github.com/kubernetes/kops/releases/download/v1.32.0-beta.1/kops-linux-amd64
```
Kubectl link for Ubuntu
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Enter the following Environmental Variables in .bashrc (all the way to bottom)

```
export NAME=mscgov.xyz
export KOPS_STATE_STORE=s3://mscgov.xyz
export AWS_REGION=us-east-1
export CLUSTER_NAME=mscgov.xyz
export EDITOR='/usr/bin/nano')
```

After copying the above Environmental variables to .bashrc run “ source .bashrc ”.

## Create Cluster
Create a Cluster using Kops and generate a cluster file 

```
kops create cluster --name=mscgov.xyz \
--state=s3://mscgov.xyz --zones=us-east-1a,us-east-1b \
--node-count=2 --control-plane-count=1 --node-size=t3.medium --control-plane-size=t3.medium \
--control-plane-zones=us-east-1a --control-plane-volume-size 10 --node-volume-size 10 \
--ssh-public-key ~/.ssh/id_ed25519.pub \
--dns-zone=mscgov.xyz --dry-run --output yaml
```

kops create -f cluster.yml

kops update cluster --name mscgov.xyz --yes --admin

kops validate cluster --wait 10m

kops delete -f cluster.yml  --yes


Below medium blog has a detailed explanation on how to create your first Kops Cluster:
https://medium.com/@csarat424/master-production-grade-kubernetes-with-kops-on-aws-the-ultimate-guide-ae3418e7d85d

Step 2 --> Create an EC2 Instance named TLS-keys (t2.micro)

Install certbot

```
sudo snap install --classic cerbot
```
```
certbot certonly --manual --preferred-challenges dns \
--manual-public-ip-logging-ok --key-type rsa --rsa-key-size 4096 \
--email xyz@gmail.com --server https://acme-v02.api.letsencrypt.org/directory \
--agree-tos -d "*.ssdgov.xyz
```

Now deploy the generated DNS Text Record in Route 53

Next Deploy the Ingress Controller

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/aws/deploy.yaml
```

