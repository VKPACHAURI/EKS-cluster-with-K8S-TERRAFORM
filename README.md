# EKS-cluster-with-K8S-TERRAFORM


🚀 Ultimate Guide: Create AWS EKS with Terraform (Step-by-Step with Explanation)





Hi I’m vishesh pachauri. Here you will Linux, AWS and DevOps related video in Hindi.

Learn how to deploy an AWS EKS cluster using Terraform in this hands-on guide. Follow step-by-step instructions to set up EKS, configure security, and manage Kubernetes workloads. Perfect for DevOps engineers and cloud professionals! 🚀

Amazon Elastic Kubernetes Service (EKS) is one of the most powerful managed Kubernetes services in the cloud. It allows businesses to deploy, scale, and manage containerized applications without worrying about the complexities of setting up and maintaining Kubernetes clusters.

But setting up AWS EKS manually can be time-consuming and error-prone. This is where Terraform, an Infrastructure as Code (IaC) tool, comes into play.

In this step-by-step guide, we’ll walk you through deploying an AWS EKS cluster using Terraform, ensuring a scalable, production-ready setup. By the end, you’ll have a fully functional EKS cluster that you can use to run Kubernetes workloads.

Zoom image will be displayed

Create AWS EKS with Terraform
Why Use Terraform for AWS EKS Deployment?
Terraform is one of the most widely adopted tools for cloud automation. Here’s why you should use it for your AWS EKS setup:

✅ Infrastructure as Code (IaC): Easily manage and automate infrastructure deployment.
✅ Scalability: Adjust configurations without manual intervention.
✅ Reproducibility: Consistently deploy identical environments across different regions.
✅ Time-Saving: Automate cluster creation and avoid repetitive tasks.

Now, let’s get started with the hands-on implementation.

Prerequisites
Before diving into Terraform scripting, ensure you have the following:

🔹 An AWS Account (with IAM user having AdministratorAccess)

🔹 AWS CLI Installed (for configuring AWS credentials)

🔹 Terraform Installed (latest version recommended: 1.6.0+)

🔹 kubectl Installed (for managing Kubernetes cluster)

🔹 eksctl Installed (optional, but useful for quick configurations)

Now, let’s move forward!

Step 1: Set Up Your AWS IAM User for Terraform
Terraform requires an AWS IAM user to provision resources. Create an IAM user with AdministratorAccess in AWS Console.

Next, configure AWS credentials on your local machine:

aws configure
Enter the following details:

AWS Access Key ID

AWS Secret Access Key

Default AWS Region (e.g., us-east-1)

Step 2: Install Required Tools
Install Terraform, kubectl, and eksctl by running these commands:

# Update package lists
sudo apt update && sudo apt install -y unzip curl awscli

# Install Terraform (latest stable version)
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform -v
# Install kubectl (Kubernetes CLI)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
Now, you’re ready to deploy EKS using Terraform.

Step 3: Set Up Terraform Files for AWS EKS
1️⃣ Create a Terraform Directory & Clone the Code Repo
mkdir terraform-eks && cd terraform-eks
git clone https://github.com/techmahato-com/terraform-project-for-beginners.git
Directory is the best practices for creating any project manifest file because you can easy manage track from here and you will get clear visivility for update and upgrade for the next time.

2️⃣ Define Terraform Version (versions.tf)
# versions.tf
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
    tls = {
      source  = "hashicorp/tls"
      version = "~> 4.0"
    }
  }
}
The version of Terraform required for the configuration is 1.6.0 or higher.
The AWS provider (from HashiCorp) is required, and its version should be 5.0 or higher.
3️⃣ Define Input Variables (variables.tf)
# variables.tf
variable "kubernetes_version" {
  default     = "1.31"
  description = "EKS Kubernetes version"
  type        = string
}

variable "vpc_cidr" {
  default     = "10.0.0.0/16"
  description = "VPC CIDR range"
  type        = string
}

variable "aws_region" {
  default     = "us-east-1"
  description = "AWS region"
  type        = string
}

variable "cluster_name" {
  default     = "techmahato-eks"
  description = "EKS cluster name"
  type        = string
}
aws_region: Sets a default value of "us-east-1" for the AWS region where resources will be created.
kubernetes_version: Sets a default value of "1.28" for the Kubernetes version to be used.
vpc_cidr: Sets a default value of "10.0.0.0/16" for the CIDR block (IP address range) of the VPC (Virtual Private Cloud).
4️⃣ Create VPC and Subnets (vpc.tf)
# vpc.tf
provider "aws" {
  region = var.aws_region
}

data "aws_availability_zones" "available" {}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.7.0"

  name = "${var.cluster_name}-vpc"
  cidr = var.vpc_cidr

  azs             = slice(data.aws_availability_zones.available.names, 0, 3)
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = 1
  }
}
source: Specifies the source of the module, which is the terraform-aws-modules/vpc/aws module from the Terraform Registry.
version: Specifies the version of the module to use (in this case, 5.7.0).
name: The name of the VPC, which is set to "eks-vpc".
cidr: Sets the CIDR block for the VPC using the value from the vpc_cidr variable (which defaults to "10.0.0.0/16").
azs: Specifies the availability zones to use for the VPC (in this case, "us-east-1a" and "us-east-1b").
private_subnets: Defines two private subnets with CIDR blocks "10.0.1.0/24" and "10.0.2.0/24".
public_subnets: Defines two public subnets with CIDR blocks "10.0.3.0/24" and "10.0.4.0/24".
enable_nat_gateway: Enables a NAT Gateway to allow private subnet instances to access the internet.
5️⃣ Deploy EKS Cluster (eks-cluster.tf)
# eks-cluster.tf
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.8.5"

  cluster_name                   = var.cluster_name
  cluster_version                = var.kubernetes_version
  cluster_endpoint_public_access = true
  enable_irsa                    = true

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    main = {
      name            = "node-group"
      instance_types  = ["t3.medium"]
      min_size        = 2
      max_size        = 6
      desired_size    = 2
      capacity_type   = "ON_DEMAND"
      ami_type        = "AL2_x86_64"
      disk_size       = 20

      # Required for security group rules
      cluster_primary_security_group_id = module.eks.cluster_primary_security_group_id

      tags = {
        Environment = "demo"
      }
    }
  }

  node_security_group_additional_rules = {
    ingress_self_all = {
      description = "Node to node all ports/protocols"
      protocol    = "-1"
      from_port   = 0
      to_port     = 0
      type        = "ingress"
      self        = true
    }
    egress_all = {
      description      = "Node all egress"
      protocol         = "-1"
      from_port        = 0
      to_port          = 0
      type             = "egress"
      cidr_blocks      = ["0.0.0.0/0"]
      ipv6_cidr_blocks = ["::/0"]
    }
  }

  tags = {
    Environment = "demo"
  }
}
source: Specifies the source of the module, which is the terraform-aws-modules/eks/aws module from the Terraform Registry.
cluster_name: Sets the name of the EKS cluster to "demo-cluster".
cluster_version: Uses the value from the kubernetes_version variable (which defaults to "1.28") to define the Kubernetes version for the EKS cluster.
subnet_ids: Specifies the private subnets for the EKS cluster, which are retrieved from the module.vpc.private_subnets output (from the previously defined VPC module).
vpc_id: Specifies the VPC ID for the EKS cluster, which is retrieved from the module.vpc.vpc_id output (from the VPC module).
6️⃣ Define Outputs (outputs.tf)
# outputs.tf

output "cluster_id" {
  description = "EKS cluster ID"
  value       = module.eks.cluster_id
}

output "cluster_endpoint" {
  description = "EKS control plane endpoint"
  value       = module.eks.cluster_endpoint
}

output "cluster_oidc_provider_arn" {
  description = "EKS OIDC provider ARN"
  value       = module.eks.oidc_provider_arn
}

output "configure_kubectl" {
  description = "Command to configure kubectl"
  value       = module.eks.cluster_id != null ? "aws eks --region ${var.aws_region} update-kubeconfig --name ${module.eks.cluster_id}" : "EKS cluster is not yet created"
}


output "node_security_group_id" {
  description = "Worker node security group ID"
  value       = module.eks.cluster_security_group_id
}

output "cluster_certificate_authority" {
  description = "EKS cluster certificate authority"
  value       = module.eks.cluster_certificate_authority_data
}

output "cluster_iam_role_name" {
  description = "IAM role name for the EKS cluster"
  value       = module.eks.cluster_iam_role_name
}

output "eks_version" {
  description = "EKS Kubernetes version"
  value       = module.eks.cluster_version
}
source: Specifies the source of the module, which is the terraform-aws-modules/eks/aws module from the Terraform Registry.
cluster_name: Sets the name of the EKS cluster to "demo-cluster".
cluster_version: Uses the value from the kubernetes_version variable (which defaults to "1.28") to define the Kubernetes version for the EKS cluster.
subnet_ids: Specifies the private subnets for the EKS cluster, which are retrieved from the module.vpc.private_subnets output (from the previously defined VPC module).
vpc_id: Specifies the VPC ID for the EKS cluster, which is retrieved from the module.vpc.vpc_id output (from the VPC module).
Step 4: Deploy Infrastructure with Terraform
Run these commands to deploy AWS EKS:

terraform init
terraform validate
terraform plan
terraform apply -auto-approve
After a few minutes, your EKS cluster will be ready!

Step 5: Configure kubectl to Access EKS
Once the cluster is deployed, configure kubectl to interact with it:

aws eks update-kubeconfig --name demo-cluster --region us-east-1
Verify that your cluster:

# Get EKS Cluster Details
aws eks describe-cluster --name demo-cluster --region us-east-1
# Example output: Shows details of the "demo-cluster" in "us-east-1"

# List Node Groups in an EKS Cluster
aws eks list-nodegroups --cluster-name demo-cluster --region us-east-1
# Example output: Lists node groups in "demo-cluster"

# Describe a Node Group
aws eks describe-nodegroup --cluster-name demo-cluster --nodegroup-name demo-nodegroup --region us-east-1
# Example output: Shows details of the "demo-nodegroup" in "demo-cluster"

# List EC2 Instances (Nodes) in the Cluster
kubectl get nodes
# Example output: Lists all the nodes in the cluster

# Get Detailed Information on a Node
kubectl describe node ip-192-168-1-10.us-east-1.compute.internal
# Example output: Shows detailed information for the specific node "ip-192-168-1-10"

# List Namespaces
kubectl get namespaces
# Example output: Lists all namespaces in the cluster

# Create a New Namespace
kubectl create namespace demo-namespace
# Example output: Creates "demo-namespace"

# Switch to a Specific Namespace
kubectl config set-context --current --namespace=demo-namespace
# Example output: Switches to the "demo-namespace"

# List Pods in a Namespace
kubectl get pods --namespace demo-namespace
# Example output: Lists all pods in the "demo-namespace"

# Get Detailed Information on a Pod
kubectl describe pod my-pod --namespace demo-namespace
# Example output: Shows detailed information for the pod "my-pod" in "demo-namespace"

# Delete a Pod
kubectl delete pod my-pod --namespace demo-namespace
# Example output: Deletes the "my-pod" pod in "demo-namespace"

# List Deployments in a Namespace
kubectl get deployments --namespace demo-namespace
# Example output: Lists all deployments in "demo-namespace"

# Create a Deployment
kubectl create deployment nginx-deployment --image=nginx --namespace demo-namespace
# Example output: Creates a deployment "nginx-deployment" with the "nginx" image in "demo-namespace"

# Get Cluster Context
kubectl config current-context
# Example output: Shows the current cluster context (e.g., "arn:aws:eks:us-east-1:123456789012:cluster/demo-cluster")

# Set EKS Cluster Context (if not already configured)
aws eks --region us-east-1 update-kubeconfig --name demo-cluster
# Example output: Configures kubectl to use the "demo-cluster" in "us-east-1"
Step 6: Check Cluster in GIU
Use AWS Management Console and check below

VPC & Subnets
EKs Cluster and Node Groups
Step 7: Cleanup & Destroy Resources
If you need to remove your EKS cluster, run:

terraform destroy -auto-approve
Step 8: Verify by AWs Console, Cluster and Resources Deleted
Use AWS Management Console and check below

VPC & Subnets
EKs Cluster and Node Groups
Conclusion
Congratulations! 🎉 You’ve successfully deployed an AWS EKS cluster using Terraform. Now you can:

✅ Deploy workloads on Kubernetes
✅ Set up Ingress Controllers & Load Balancers
✅ Enable Auto-scaling & Monitoring

🔹 Next Steps: Integrate Terraform with CI/CD pipelines for automated deployments.

🚀 Share this guide if you found it helpful!
