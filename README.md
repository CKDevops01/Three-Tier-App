# Three-Tier-App

# Overview

Deploying a Three-Tier Web Application using ReactJS, NodeJS, and MongoDB, with deployment on AWS EKS.Three-Tier -App(React jS+node jS+mongodb) application deployment.

# Getting Started

# Step 1: IAM Configuration

Create a user eks-admin with AdministratorAccess.
Generate Security Credentials: Access Key and Secret Access Key.

# Step 2: EC2 Setup

Launch an Ubuntu instance in your favourite region (eg. region ap-south-1).
SSH into the instance from your local machine.

# Step 3: Install AWS CLI v2

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```


# Step 4: Install Docker

```bash
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

# Step 5: Install kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

# Step 6: Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

# Step 7: Setup EKS Cluster

```bash
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region ap-south-1 --name three-tier-cluster
kubectl get nodes
```

# Step 8: Install ArgoCD 
-*Create seperate namespace for argocd*
```ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
```
-*Expose argocd-server by loadbalancer*
```ArgoCD
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname'`
```
-*Expose password of argocd-server*
```ArgoCD
export ARGO_PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
```

```
>kubectl create namespace three-tier
```
# Step 9: Install AWS Load Balancer

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<aws_account_id>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=ap-south-1
```
# Step 10: Deploy AWS Load Balancer Controller

```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
#kubectl apply -f full_stack_lb.yaml
```
# Cleanup
To delete the EKS cluster:
```bash
eksctl delete cluster --name three-tier-cluster --region ap-south-1
```
