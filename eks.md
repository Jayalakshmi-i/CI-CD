# Deploying 2048 Game application on AWS EKS using Ingress and ALB Ingress Controller

This project showcases how to deploy real-time 2048 game application on AWS EKS, which have access through the Application load balancer and then expose it to outside world using Ingress and ALB Ingress Controller.

**Prerequisites:**

1. **kubectl** – Command line utility used to manage the Kubernetes clusters. It is the main interface that allows users to create (and manage) individual objects or groups of objects inside a Kubernetes cluster.  
2. **eksctl** – Amazon's managed Kubernetes service for EC2. A command line tool for creating and managing clusters on EKS that automates many individual tasks.  
3. **AWS CLI** – A command line tool for working with AWS services, including Amazon EKS. The AWS Command Line Interface (CLI) provides a unified tool to manage your AWS services directly from the command line. After installing the AWS CLI, it is recommended to configure it.  So, we can control multiple AWS services from the command line and automate them through scripts.

**Components and Tools used:**

* **EKS Cluster:** The foundation for our 2048 Game application. This is our Kubernetes environment managed by AWS.  
* **Fargate Profile:** This allows users to focus on building applications without managing servers and to declare which Pods run on Fargate.  
* **Namespace**: A virtual cluster within Kubernetes, which helps isolate our resources logically. The 2048 game will reside in the game-2048 namespace.  
* **Ingress**: Routes the traffic inside the cluster. Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.  
* **AWS Load Balancer Controller**: This is the Kubernetes controller that integrates with AWS Application Load Balancers (ALBs). It reads the game-2048 ingress resource and creates an Application Load Balancer with all the necessary configurations, such as target groups and the port the pods should listen to.  
* **IAM OIDC Provider:** OIDC can be used as an identity provider for EKS clusters, allowing you to authenticate and authorize users to access Kubernetes resources. Here we use the IAM identity provider to integrate with other services.  
* **Deployment & Service:** A YAML file is created to define deployments, services, and ingress to define replicas/instances, application port details, and the type of service to deploy the 2048 game application.

**Steps to Deployment**

**Step 1: Create an EKS cluster using eksctl**

eksctl create cluster \--name demo-cluster \--region us-east-1 –fargate

The command will automatically create entire cluster with various resources like Fargate instances, VPC, private and public subnets, route tables, NAT gateways, and IAM service roles.

![][image1]

On the successful execution of eksctl command, we can see the EKS cluster and Worker Nodes created under Compute.

![][image2]

![][image3]

![][image4]

**Configuring kubectl for EKS**: To control our EKS cluster using kubectl commands it needs to update our local kubeconfig file to connect to an Amazon EKS Cluster named \`demo-cluster\`, which configures our kubectl by creating or updating the kubeconfig.

![][image5]

**Step 2: Creating Fargate profile with the namespace ‘game-2048’ to deploy 2048 application pods:**

![][image6]

![][image7]

**Step 3: Deploying the 2048 Gaming Application with YAML configurations files**

Create a YAML file to define deployments, services, and ingress for your application.

We can define deployments, services and ingress resources separately or everything in one yaml file with the ports and application image depending on your requirement.

**Deployment.yaml**

* Created a namespace (game-2048).  
* Created Deployment in game-2048 namespace.  
* Created Labels and Selectors with the name ‘app-2048’, so the service can identify our pods. Note: The labels and selector names are used as in the YAML file.  
* Defined number of pod replicas for this deployment.  
* Update the Image with the latest application's docker image.  
* update the container port.

![][image8]

**Service.yaml**

* Used game-2048 namespace for this service.   
* Update application port and container Port details.  
* Defined ‘NodePort’  service type mode.   
* Mentioned ‘app-2048’ as selector to for the set of pods on which we want to apply this service.

![][image9]

**Ingress.yaml**

* Mention the namespace for this Ingress.  
* Annotations like internet-facing or intranet-facing. In this case, we want the Ingress to be internet facing so our application inside the pods can be accessed from outside world.  
* Mentioned the name of the service that this Ingress is going to use.

![][image10]

All the above configuration for deployments, services and Ingress are mentioned in the file: [https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048\_full.yaml](https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml)

So, we can apply this file configuration to create deployments, services and Ingress resources for the 2048 game application using the ‘kubectl apply’ command.

![][image11]

All the above configurations are applied successfully:

![][image12]

![][image13]

![][image14]

Here we can notice that, we do not have the address in the ingress to access it from the outside world. This is because we did not configure ingress controller. 

This ingress controller will read this ingress-2048 ingress resource and create Application LB for us with all configurations like target groups and what port the pods should listen

Before deploying this ingress ALB controller, we need access to AWS account so need to create a IAM OIDC provider so that ALB within AWS account can be integrated to gain access

**Step 4:** **Configure IAM OIDC Provider and IAM Policy:**

Now Associate an IAM OIDC provider with your cluster:

![][image15]

![][image16]

Create an IAM policy for the AWS Load Balancer Controller:

![][image17]

Now whenever a pod is running that will have service account and for that service account, we need IAM role attached so that it can talk to other services in the AWS account

Attaching ‘*AmazonEKSLoadBalancerControllerRole*’ role to the service account of pod

![][image18]

**Step 5: Installing Helm Charts for ALB Controller:**

Adding EKS to Helm repositories, and update the Helm repo, and then install the ALB controller using Helm charts.

![][image19]

![][image20]

**Step 6: Deploy AWS Load Balancer Controller:**

Helm chart will create actual controller and it will use this service account for running the pod

![][image21]

Successfully deployed 2 replicas of ALB controller each in different availability zones.

![][image22]

![][image23]

![][image24]

The above ALB controller successfully created an Application Load Balancer using the ingress resource that created earlier.

![][image25]

Here we got the LB address 

![][image26]

![][image27]

**Step 7: Verifying and accessing the Application**

Accessing the game 2048 application using LB address

![][image28]
