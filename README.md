# Project Title: Three-Tier Web Application using ReactJS, NodeJS, and MongoDB, with Deployment on AWS EKS

## Introduction
This project demonstrates the development and deployment of a three-tier web application. The application consists of a front-end, a back-end, and a database layer, each implemented using modern web technologies and deployed on AWS Elastic Kubernetes Service (EKS).

## Technologies Used
- Front-End: ReactJS
- Back-End: NodeJS
- Database: MongoDB
- Containerization: Docker
- Orchestration: Kubernetes (EKS)
- Cloud Platform: AWS

## Prerequisites
- Basic knowledge of Docker, and AWS services.
- An AWS account with necessary permissions.
- Knowledge on K8s and EKS service.


## Application Code
The `Application-Code` directory contains the source code for the Three-Tier Web Application. Dive into this directory to explore the frontend and backend implementations.

## Jenkins Pipeline Code
In the `Jenkins-Pipeline-Code` directory, you'll find Jenkins pipeline scripts. These scripts automate the CI/CD process, ensuring smooth integration and deployment of your application.

## Jenkins Server Terraform
Explore the `Jenkins-Server-TF` directory to find Terraform scripts for setting up the Jenkins Server on AWS. These scripts simplify the infrastructure provisioning process.

## Kubernetes Manifests Files
The `Kubernetes-Manifests-Files` directory holds Kubernetes manifests for deploying your application on AWS EKS. Understand and customize these files to suit your project needs.

## Project Details
🛠️ **Tools Explored:**
- Terraform & AWS CLI for AWS infrastructure
- Jenkins, Sonarqube, Terraform, Kubectl, and more for CI/CD setup
- Helm, Prometheus, and Grafana for Monitoring
- ArgoCD for GitOps practices

🚢 **High-Level Overview:**
- IAM User setup & Terraform magic on AWS
- Jenkins deployment with AWS integration
- EKS Cluster creation & Load Balancer configuration
- Private ECR repositories for secure image management
- Helm charts for efficient monitoring setup
- GitOps with ArgoCD - the cherry on top!

📈 **The journey covered everything from setting up tools to deploying a Three-Tier app, ensuring data persistence, and implementing CI/CD pipelines.**

## Getting Started
To get started with this project, refer to [comprehensive guide](https://amanpathakdevops.medium.com/advanced-end-to-end-devsecops-kubernetes-three-tier-project-using-aws-eks-argocd-prometheus-fbbfdb956d1a) that walks you through IAM user setup, infrastructure provisioning, CI/CD pipeline configuration, EKS cluster creation, and more.

### Step1:
- create EC2 Instance (ubuntu)  and clone the github repo in it.
- Install docker on instance.
```
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### Step 2: Install AWS CLI v2 and Configure the AWS account
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure  ( before the aws configure, create the IAM user and create access key from it)
```

### Step 3: IAM Configuration
- Create a user `eks-admin` with `AdministratorAccess`.
- Generate Security Credentials: Access Key and Secret Access Key.

### Step 4: Write the docker file, create image and push to ECR.
- write dockerfile for the backend and frontend.
- now push the image to ECR:
- ```go to ECR on AWS Console - create repo - public/private - repo name - create```
- ```now click on repo and click on 'view push command' and follow the steps/ command given.```
- create the two ECR repo for the frontend and backend, and follow stpes for the both.
```
- Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI: ( command to login into ECR)
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/y7n6g2o6
(Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.)

- Build your Docker image using the following command. For information on building a Docker file from scratch see the instructions here . You can skip this step if your image is already built:
docker build -t three-tier-frontend .

- After the build completes, tag your image so you can push the image to this repository:
docker tag three-tier-frontend:latest public.ecr.aws/y7n6g2o6/three-tier-frontend:latest

- Run the following command to push this image to your newly created AWS repository:
docker push public.ecr.aws/y7n6g2o6/three-tier-frontend:latest 
```
### Step 5: Install kubectl
``` shell
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Step 6: Install eksctl
``` shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Step 7: Setup EKS Cluster

``` shell
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```
once above command will execute you will see the to ec2 instance or node has been created. (t2.medium)
you can check the created cluster on AWS console also you and can check on Cloud Formation as well.

### Step 8: Create and Run Manifests.
- for Mongodb
- ```kubectl create namespace workshop```
```
kubectl apply -f deployment.yaml
kubectl apply -f secrets.yaml
kubectl apply -f pvc.yaml
kubectl apply -f pv.yaml
kubectl apply -f service.yaml
```
- for backend
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
- for frontend
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Step 9: Install AWS Load Balancer

- lets download the AWS LB IAM policy:(this policy will allow the external traffic from LB to K8s, This command downloads the IAM policy file needed for the AWS Load Balancer Controller)

```curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json```

- creates a new IAM policy named AWSLoadBalancerControllerIAMPolicy using the downloaded JSON file ( to connect EKS ans LB)

```aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json```

- associates the IAM OIDC provider with your EKS cluster named three-tier-cluster in the us-east-1 region, enabling the cluster to use IAM roles for service accounts.

```eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=three-tier-cluster --approve```

- This command creates an IAM service account named aws-load-balancer-controller in the kube-system namespace of your EKS cluster.

  It attaches the previously created IAM policy (AWSLoadBalancerControllerIAMPolicy) to a new IAM role (AmazonEKSLoadBalancerControllerRole)

```
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::806956591980:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1
```


### Step 10: Deploy AWS Load Balancer Controller
``` shell
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f ingress.yaml
```

### Check Successfull Deployment

```
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$ kubectl get pods -n three-tier
NAME                        READY   STATUS    RESTARTS   AGE
api-54f9fbc8bb-9r8nf        1/1     Running   0          68m
frontend-64fd5bbfb9-cwqfd   1/1     Running   0          66m
mongodb-6f57ffcc44-pl5xz    1/1     Running   0          73m
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$
```
```
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$ kubectl get deploy -n three-tier
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
api        1/1     1            1           68m
frontend   1/1     1            1           66m
mongodb    1/1     1            1           73m
```
```
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$ kubectl get svc -n three-tier
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
api           ClusterIP   10.100.143.241   <none>        3500/TCP    68m
frontend      ClusterIP   10.100.95.77     <none>        3000/TCP    67m
mongodb-svc   ClusterIP   10.100.141.162   <none>        27017/TCP   73m
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$
```
```
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$ kubectl get ing -n three-tier
NAME     CLASS   HOSTS   ADDRESS                                                                 PORTS   AGE
mainlb   alb     *       k8s-threetie-mainlb-73c240e86c-1660525476.us-east-1.elb.amazonaws.com   80      53m
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$
```
``` 
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$ kubectl get deployment -n kube-system aws-load-balancer-controller
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           59m
ubuntu@ip-172-31-23-184:~/TWSThreeTierAppChallenge/Kubernetes-Manifests-file$
```
### Cleanup
- To delete the EKS cluster:
``` shell
eksctl delete cluster --name three-tier-cluster --region us-west-2
```


