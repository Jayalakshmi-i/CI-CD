# Deploying 2048 Game application on AWS EKS using Ingress and ALB Ingress Controller

This project showcases how to deploy a real-time 2048 game application on AWS EKS, which has access through the Application load balancer and then expose it to the outside world using Ingress and ALB Ingress Controller.

**Prerequisites:**

1. **kubectl** – Command line utility used to manage the Kubernetes clusters. It is the main interface that allows users to create (and manage) individual objects or groups of objects inside a Kubernetes cluster.  
2. **eksctl** – Amazon's managed Kubernetes service for EC2. A command-line tool for creating and managing clusters on EKS that automates many individual tasks.  
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

```eksctl create cluster \--name demo-cluster \--region us-east-1 –fargate```

The command will automatically create an entire cluster with various resources like Fargate instances, VPC, private and public subnets, route tables, NAT gateways, and IAM service roles.

![image](https://github.com/user-attachments/assets/9cdca39b-4d93-45d4-8dcb-208f25722be9)


On the successful execution of the eksctl command, we can see the EKS cluster and Worker Nodes created under Compute.

![image](https://github.com/user-attachments/assets/7cee5177-64a7-4ec1-873e-9e186f688f65)


![image](https://github.com/user-attachments/assets/93fdd9bc-576b-4a75-a648-a7b3a73f76a0)


![image](https://github.com/user-attachments/assets/5dd751e7-3c53-490d-8aec-cb8b9277e81b)


**Configuring kubectl for EKS**: To control our EKS cluster using kubectl commands it needs to update our local kubeconfig file to connect to an Amazon EKS Cluster named \`demo-cluster\`, which configures our kubectl by creating or updating the kubeconfig.

```
bash
aws eks update-kubeconfig --name demo-cluster --region us-east-1

```

**Step 2: Creating Fargate profile with the namespace ‘game-2048’ to deploy 2048 application pods:**

![image](https://github.com/user-attachments/assets/9616137c-53dc-4341-b7ef-bf65b93e4c82)

**Step 3: Deploying the 2048 Gaming Application with YAML configurations files**

Create a YAML file to define deployments, services, and ingress for your application.

We can define deployments, services and ingress resources separately or everything in one yaml file with the ports and application image depending on your requirement.

**Deployment.yaml**

* Created a namespace (game-2048).  
* Created Deployment in game-2048 namespace.  
* Created Labels and Selectors with the name ‘app-2048’, so the service can identify our pods. Note: The labels and selector names are used as in the YAML file.  
* Defined the number of pod replicas for this deployment.  
* Update the Image with the latest application's docker image.  
* update the container port.

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: game-2048
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: game-2048
  name: deployment-2048
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: app-2048
  replicas: 5
  template:
    metadata:
      labels:
        app.kubernetes.io/name: app-2048
    spec:
      containers:
      - image: public.ecr.aws/l6m2t8p7/docker-2048:latest
        imagePullPolicy: Always
        name: app-2048
        ports:
        - containerPort: 80
```

**Service.yaml**

* Used game-2048 namespace for this service.   
* Update application port and container Port details.  
* Defined ‘NodePort’  service type mode.   
* Mentioned ‘app-2048’ as a selector to for the set of pods on which we want to apply this service.

```
---
apiVersion: v1
kind: Service
metadata:
  namespace: game-2048
  name: service-2048
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: NodePort
  selector:
    app.kubernetes.io/name: app-2048
```

**Ingress.yaml**

* Mention the namespace for this Ingress.  
* Annotations like internet-facing or intranet-facing. In this case, we want the Ingress to be internet facing so our application inside the pods can be accessed from outside world.  
* Mentioned the name of the service that this Ingress is going to use.

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: game-2048
  name: ingress-2048
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: service-2048
              port:
                number: 80
```

All the above configurations for deployments, services and Ingress are mentioned in the file: [https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048\_full.yaml](https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml)

We can use the `kubectl apply` command to apply this file configuration to create deployments, services, and Ingress resources for the 2048 game application.

![image](https://github.com/user-attachments/assets/c34f416e-2152-4e7a-ab69-425407cdfe47)


All the above configurations are applied successfully:

![image](https://github.com/user-attachments/assets/eb3ed3d3-e3e3-44f5-90bc-40482cb99242)

![image](https://github.com/user-attachments/assets/84bcfff9-051f-431c-9414-a4bd03f829ca)

![image](https://github.com/user-attachments/assets/43e479b9-5c66-4db4-8755-abe6a2fd18f4)


Here we can notice that, we do not have the address in the ingress to access it from the outside world. This is because we did not configure ingress controller. 

This ingress controller will read this ingress-2048 ingress resource and create Application LB for us with all configurations like target groups and what port the pods should listen

Before deploying this ingress ALB controller, we need access to the AWS account so need to create an IAM OIDC provider so that ALB within the AWS account can be integrated to gain access

**Step 4:** **Configure IAM OIDC Provider and IAM Policy:**

next, associating an IAM OIDC provider with the cluster:

![image](https://github.com/user-attachments/assets/6faa1704-437e-45ae-a978-40dd3c73d540)

![image](https://github.com/user-attachments/assets/318e4560-b040-44f3-b460-7b496b78b9cc)

Create an IAM policy for the AWS Load Balancer Controller:

![image](https://github.com/user-attachments/assets/92b72d74-88b8-4034-8541-4648e9d73f8d)

Now whenever a pod is running it will have a service account and for that service account, we need the IAM role attached so that it can talk to other services in the AWS account

Attaching the ‘*AmazonEKSLoadBalancerControllerRole*’ role to the service account of the pod

```
eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
  ```
  

**Step 5: Installing Helm Charts for ALB Controller:**

Adding EKS to Helm repositories, updating the Helm repo, and then installing the ALB controller using Helm charts.

```
helm repo add eks https://aws.github.io/eks-charts
```

![image](https://github.com/user-attachments/assets/5844394d-19ee-49ef-bf07-714445f5c1dc)

**Step 6: Deploy AWS Load Balancer Controller:**

The helm chart will create the actual controller and it will use this service account to run the pod

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>

```
  ![image](https://github.com/user-attachments/assets/a0f05d66-ff6c-4988-ae44-0ac395885fa0)

Successfully deployed 2 replicas of the ALB controller each in different availability zones.

![image](https://github.com/user-attachments/assets/25188da1-2c01-49fc-b399-0a4f2801d110)

![image](https://github.com/user-attachments/assets/2b1418af-6795-4be7-84ef-a2637e3631dc)

![image](https://github.com/user-attachments/assets/0b54da12-d6d1-43bc-a9d7-98ce19cde506)


The above ALB controller successfully created an Application Load Balancer using the ingress resource that was created earlier.

![image](https://github.com/user-attachments/assets/064bb560-aec5-4f6d-8167-8e5486beebb4)

Here we got the LB address 

![image](https://github.com/user-attachments/assets/5b4dd370-5e6c-41d5-ad15-24d6b348e151)


**Step 7: Verifying and accessing the Application**

Accessing the game 2048 application using the LB address

![image](https://github.com/user-attachments/assets/0c38f753-8802-4056-973a-1f28a15e7074)

