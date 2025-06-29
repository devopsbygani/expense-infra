# Expense DEV Infrastructure using Terraform With deployments on Kubernetes k8.
# This creates Infra as mentioned below.
* VPC -Virtual private CLoud (vpc,igw,nat gateway,elasticip,public subnet,private subnet,database subnet,databasesubent group,route tables,routes and route association)
* SG -security group and Security group rules
* Bastion server ( for connecting to nodes and creating microservices deployment)
* RDS - ( realtional Database Manager) mysql 
* ACM - Amazon certificate manager.
* EKS - elastic Kubernetes service .
* ALB - Application Load balancer 

## 🔧 Infrastructure Requirements
Ensure all resources are created with the suffix `-dev` to prevent conflicts with production resources.
---

## Bastion Host Configuration
The Bastion Host will act as the EKS client. Install the following tools on the bastion instance:

- `kubectl`
- `helm`
- `aws configure`
- `k9s`
- RDS CLI (e.g., `mysql`)

---
##  RDS MySQL Setup

* We are using **Amazon RDS** instead of a MySQL container. Schema is already created via RDS provisioning, but you need to manually create the **table** and **user**.
* Make sure the MySQL instance allows **port 3306** from the Bastion host.

### Connect to MySQL
```bash
mysql -h expense-dev.czn6yzxlcsiv.us-east-1.rds.amazonaws.com -u root -pExpenseApp1
```

### Create Table and User
```sql
USE transactions;

CREATE TABLE IF NOT EXISTS transactions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    amount INT,
    description VARCHAR(255)
);

CREATE USER IF NOT EXISTS 'expense'@'%' IDENTIFIED BY 'ExpenseApp@1';
GRANT ALL ON transactions.* TO 'expense'@'%';
FLUSH PRIVILEGES;
```
## Frontend Setup

Make sure:

- `nginx.conf` listens on port `8080`
- ALB **Target Group Health Check Port** = `8080`
- Kubernetes frontend **Service targetPort** = `8080`

---
### Update Kubeconfig from Bastion
```bash
aws configure
aws eks update-kubeconfig --region us-east-1 --name expense-dev
kubectl get nodes
```

### 📦 Create Namespace

```bash
kubectl create namespace expense
```

---

## 5️⃣ AWS Load Balancer Controller Installation

### 📁 Create IAM Policy

```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.10.0/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

### 👤 Create IAM Service Account

Replace the AWS Account ID `315069654700` with your actual ID.

```bash
eksctl create iamserviceaccount \
--cluster=expense-dev \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::315069654700:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

### ☸️ Install Controller using Helm

```bash
helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=expense-dev \
  --set serviceAccount.create=true \
  --set serviceAccount.name=aws-load-balancer-controller
```

---

## ✅ Validate Pods

Check if Load Balancer controller and other system pods are running:

```bash
kubectl get pods -n kube-system
```

### 📋 Sample Output:

```text
NAME                                            READY   STATUS    RESTARTS   AGE
aws-load-balancer-controller-689495d45f-mwmg6   0/1     Running   0          8s
aws-load-balancer-controller-689495d45f-v78wh   0/1     Running   0          8s
aws-node-txwjc                                  2/2     Running   0          5m13s
aws-node-v9d79                                  2/2     Running   0          5m17s
coredns-789f8477df-55j2d                        1/1     Running   0          9m52s
coredns-789f8477df-74j5h                        1/1     Running   0          9m52s
eks-pod-identity-agent-9ngdt                    1/1     Running   0          5m17s
eks-pod-identity-agent-cj98g                    1/1     Running   0          5m17s
kube-proxy-7sgw7                                1/1     Running   0          6m4s
kube-proxy-8zb7z                                1/1     Running   0          6m5s
```

---

## 📘 Summary

- ✅ All resources should use `-dev` suffix
- ✅ MySQL table and user setup done manually via RDS
- ✅ Frontend uses port 8080 for compatibility with non-root containers
- ✅ Ingress controller setup includes IAM role, service account, and Helm installation
- ✅ Bastion host has all required tools to access and manage EKS and RDS

---