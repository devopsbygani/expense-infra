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