Practicing Terraform Locally Without a Cloud Provider
#
devops
#
terraform
#
kubernetes
#
docker
Practicing Terraform locally offers several advantages, especially for those who are new to infrastructure as code (IaC) or want to experiment without incurring cloud costs. Here are some key benefits:

Cost-Effective Learning: You can experiment and learn Terraform without the need for cloud resources, which can be costly.
Rapid Iteration: Local environments allow for quick testing and iteration, speeding up the learning process.
Isolated Environment: A local setup provides a sandboxed environment where you can make mistakes without affecting production systems.
No Cloud Dependencies: You can practice Terraform without needing access to cloud providers, making it accessible to everyone.
Realistic Environment: Using tools like kind (Kubernetes IN Docker) allows you to simulate a real Kubernetes cluster locally.
Before diving into Terraform configurations, let's start by setting up a local Kubernetes cluster using kind (Kubernetes IN Docker). This will provide the foundation for our Terraform practice environment.

Creating a Kind Cluster
Kind is a lightweight tool for running local Kubernetes clusters using Docker containers as nodes. Here's how to get started:

1. Install Kind
If you haven't already installed kind, you'll need to do so first:

For macOS (using Homebrew):

brew install kind
For Linux:

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
For Windows (using Chocolatey):

choco install kind
2. Create a Basic Kind Configuration File
Create a file named kind-config.yaml with the following content:

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
3. Create the Cluster
Now, create your kind cluster using the configuration file:

kind create cluster --name terraform-practice --config kind-config.yaml
You should see output similar to:

Creating cluster "terraform-practice" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-terraform-practice"
You can now use your cluster with:

kubectl cluster-info --context kind-terraform-practice

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
4. Verify Your Cluster
Ensure that your cluster is running and kubectl is properly configured to use it:

kubectl cluster-info --context kind-terraform-practice
You should see information about your Kubernetes control plane and CoreDNS.

Also, check that your nodes are ready:

kubectl get nodes
The output should show your node with status "Ready".

Terraform Configuration Files
Now that your kind cluster is up and running, you can proceed with your Terraform configurations. Here are the files you need:

1. main.tf - Provider Configuration
terraform {
  required_providers {
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
}

provider "kubernetes" {
  config_path = "~/.kube/config"
}
2. nginx-deployment.tf - NGINX Deployment Configuration
resource "kubernetes_deployment" "nginx" {
  metadata {
    name = "nginx-deployment"
    labels = {
      app = "nginx"
    }
  }

  spec {
    replicas = 3

    selector {
      match_labels = {
        app = "nginx"
      }
    }

    template {
      metadata {
        labels = {
          app = "nginx"
        }
      }

      spec {
        container {
          image = "nginx:latest"
          name  = "nginx"
          port {
            container_port = 80
          }
        }
      }
    }
  }
}
3. nginx-service.tf - Service Configuration for Exposing NGINX
resource "kubernetes_service" "nginx" {
  metadata {
    name = "nginx-service"
  }

  spec {
    selector = {
      app = "nginx"
    }

    port {
      port        = 80
      target_port = 80
      node_port   = 30080  
    }

    type = "NodePort"
  }
}
Deploying with Terraform
With your kind cluster ready, you can now follow these steps to deploy your infrastructure:

Initialize Terraform:
   terraform init
Plan your deployment:
   terraform plan
Apply the configuration:
   terraform apply
Verify the deployment:
   kubectl get deployments
   kubectl get pods
   kubectl get services
Accessing Your NGINX Service
You can manually forward the port by running:

kubectl port-forward svc/nginx-service 8080:80

Then, visit:

➡ http://localhost:8080

You should see the default NGINX welcome page.

Cleaning Up
When you're done practicing, you can:

Destroy your Terraform resources:
   terraform destroy
Delete your kind cluster:
   kind delete cluster --name terraform-practice
This complete workflow gives you a practical way to experiment with Terraform locally using a kind Kubernetes cluster, without any cloud provider costs or dependencies.

Update: there some error that not allow pod's start.........blame it to the Taints

https://lnkd.in/eWwB2cEF

Apropos: 
I tried this post content on my kub on-premises .... Ubuntu 22.04.5, and I have some problems to initiate the pods...stays in PENDING state

Solution:
Got to include Taints.
Add toleration to allow pods to be scheduled on nodes with taints 

In the file nginx-deployment.yaml

toleration { 
key = "https://lnkd.in/eC4Mb3uR" 
operator = "Exists" 
effect = "NoSchedule" } 
}

You can add more tolerances if necessary for other taints.

