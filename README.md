# Deploying Microservices on AWS EKS with GitOps Workflow
<img width="1160" height="761" alt="GitOpsssss drawio" src="https://github.com/user-attachments/assets/80e31dee-8ded-4863-95a5-9cf964d255e6" />


## Project Description
This project demonstrates the deployment of a microservices-based E-Commerce application consisting of web, catalogue, cart, user, payment, shipping, ratings, dispatch, MongoDB, MySQL, Redis, and RabbitMQ services on a Kubernetes cluster. GitHub Actions is used to implement independent CI pipeline for each microservice, Helm Charts define the Kubernetes deployments, and ArgoCD automatically syncs all image updates from the GitHub Repository and deploys them to the cluster. Prometheus and Grafana are also integrated for metrics collection and visualization, enabling monitoring of the deployed microservices.

---

## Prerequisites
Before deploying this project, ensure you have the following tools installed and configured:-
- An **AWS account** with admin or equivalent permissions   
- **AWS CLI** to manage AWS resources. Make sure it is configured with credentials that have sufficient permissions.
- **eksctl** to create and manage your EKS cluster.
- **kubectl** to interact with your Kubernetes cluster.
- **Helm** to deploy and manage Helm charts for your microservices.
- **ArgoCD** to automatically sync Helm chart changes to the cluster and manage GitOps workflows.
- **AWS Load Balancer Controller (ALB Ingress Controller)** to expose services through an Application Load Balancer in your EKS cluster.
---

## Deployment Steps
### 1. Clone the Repository
```bash
git clone https://github.com/Mo7iee/GitOps-EKS.git
```
### 2. Test the app by running it locally without Kubernetes
```bash
docker-compose up
```
Once the containers are running, you can access the web service at: http://localhost:8080
### 3. Create an EKS cluster
```bash
eksctl create cluster --name demo-cluster --region us-east-1
```
### 4. Setup an ALB Ingress Controller
1. Create an IAM OIDC provider for the cluster:
```bash
eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster demo-cluster --approve
```
2. Create an IAM policy for the ALB Ingress Controller:
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
3. Create a Kubernetes service account and attach the IAM policy:
```bash
eksctl create iamserviceaccount \
  --cluster demo-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
4. Install the ALB Ingress Controller using Helm:
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=demo-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller
```
### 5. Deploy the application using Helm
```bash
helm install my-robotshop ./helm/robotshop-chart
```
You can check the status of your pods:
```bash
kubectl get pods
```
### 6. Access the Web Application
1. Get the ALB hostname:
```bash
kubectl get ingress web-ingress
```
2. Open the web application in your browser:
```bash
http://<ALB_HOSTNAME>
```
### 7. Continuous Deployment with ArgoCD
1. Install ArgoCD in the cluster:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
2. Access the Argo CD UI (LoadBalancer Service):
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
3. Get the LoadBalancer Service IP and access it from your browser:
```bash
kubectl get svc argocd-server -n argocd
```
4. Create an ArgoCD application and configure it as follows:
<!-- empty line for spacing -->
![f4d6c264-85e6-47f4-89f7-81901a268b8f](https://github.com/user-attachments/assets/915e0652-dfa7-4694-9d35-15387a3d0457)

![9104871d-5914-45cc-94fe-e56839ff384e](https://github.com/user-attachments/assets/7fa3ebc4-ec91-4ab5-819c-da4bfcb8bbba) 
### 8. Check application status on ArgoCD
You should see all application Pods and Services up and running <br><br>
![13b81bb1-69e3-4d11-8fd5-f1877bc295f1](https://github.com/user-attachments/assets/1157ae00-2748-41ae-ad80-7e1b70c8e6e3)
Now, ArgoCD will automatically sync the Helm chart whenever you push changes to your GitHub repository, ensuring automated, GitOps-style deployment.




