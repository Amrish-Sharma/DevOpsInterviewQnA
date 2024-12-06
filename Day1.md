Here are the answers to your questions:

---

### **AWS**

1. **You are unable to SSH EC2 instance in a public subnet. What could be the issue?**
   - **Security Group**: Check if the security group attached to the EC2 instance allows inbound traffic on port 22 (SSH) from your IP.
   - **Network ACL**: Verify that the subnet's Network ACL allows inbound and outbound traffic for port 22.
   - **Key Pair**: Ensure you are using the correct private key corresponding to the EC2 instance's key pair.
   - **Public IP**: Verify that the instance has a public IP or Elastic IP assigned.
   - **Routing**: Check if the route table associated with the subnet has an internet gateway.

2. **Design a highly available and scalable 3-tier architecture in AWS.**
   - **Presentation Layer**: Use an Application Load Balancer (ALB) distributing traffic to multiple EC2 instances in an Auto Scaling Group across multiple Availability Zones (AZs).
   - **Logic Layer**: Use another set of EC2 instances in an Auto Scaling Group, potentially behind a private ALB or NLB.
   - **Database Layer**: Deploy an RDS instance (Aurora or MySQL/PostgreSQL) in Multi-AZ configuration. Use Read Replicas for scalability.
   - **Networking**: Use private subnets for the logic and database layers, and public subnets for the presentation layer.

3. **How to block traffic from a particular country/region?**
   - Use **AWS WAF (Web Application Firewall)** to create a geo-matching rule that blocks traffic from the desired country/region.
   - Alternatively, use third-party services like Cloudflare with geolocation-based blocking.

4. **Your primary region suddenly goes down. How to move the application to the Disaster Recovery region?**
   - Use **Route 53 DNS failover** to route traffic to the DR region automatically.
   - For storage, use **S3 cross-region replication** or **RDS read replica promotion** in the DR region.
   - For infrastructure, use **CloudFormation** or **Terraform** to replicate resources.

5. **Write a policy that lists access to EC2 instances and S3 buckets.**
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "ec2:DescribeInstances",
                   "s3:ListBucket"
               ],
               "Resource": "*"
           }
       ]
   }
   ```

6. **Lambda function is unable to access a database (any) hosted on EC2 instance. What could be the issue?**
   - **VPC Configuration**: Ensure the Lambda function is configured to run in the same VPC as the EC2 instance.
   - **Security Groups**: Check that the Lambda function's security group allows outbound traffic to the EC2 instance's security group on the database port (e.g., 3306 for MySQL).
   - **Network Configuration**: Verify that the database instance's private IP is accessible and there is no issue with Network ACLs.

7. **Database is in a private subnet. What is the secure way to download required packages for the database?**
   - Use a **NAT Gateway** or **NAT Instance** in a public subnet to route internet-bound traffic securely.
   - Alternatively, use **AWS Systems Manager Session Manager** to manage and download packages without opening internet access.

---

### **Terraform**

1. **Write a script to create an EC2 instance and an S3 bucket.**
   ```hcl
   resource "aws_instance" "example" {
       ami           = "ami-12345678"
       instance_type = "t2.micro"
   }

   resource "aws_s3_bucket" "example" {
       bucket = "example-bucket"
   }
   ```

2. **Write a module to create an EC2 instance.**
   - **Module File** (`main.tf`):
     ```hcl
     variable "ami" {}
     variable "instance_type" {}

     resource "aws_instance" "example" {
         ami           = var.ami
         instance_type = var.instance_type
     }
     ```
   - **Usage**:
     ```hcl
     module "ec2" {
         source        = "./path-to-module"
         ami           = "ami-12345678"
         instance_type = "t2.micro"
     }
     ```

3. **Where do you keep your state file and why?**
   - Store the state file in a **remote backend** like S3 with versioning enabled for better collaboration and disaster recovery.

4. **What is a workspace in Terraform?**
   - Workspaces allow managing multiple state files from the same configuration. It is useful for isolating environments like `dev`, `staging`, and `prod`.

5. **What is state locking in Terraform?**
   - State locking prevents simultaneous operations on a state file, avoiding conflicts. Remote backends like S3 with DynamoDB provide state locking.

---

### **Kubernetes**

1. **Explain the architecture of Kubernetes.**
   - **Master Components**:
     - `API Server`: Exposes the Kubernetes API.
     - `Controller Manager`: Manages controllers like node and replica controllers.
     - `Scheduler`: Assigns pods to nodes.
     - `etcd`: Stores cluster configuration and state.
   - **Node Components**:
     - `Kubelet`: Manages containers on the node.
     - `Kube-Proxy`: Manages networking and forwarding rules.
     - `Container Runtime`: Runs containers (e.g., Docker, CRI-O).

2. **Explain the end-to-end pipeline (all stages).**
   - **Code** → **Build** → **Push to Repository** → **Deploy to Cluster** using CI/CD tools like Jenkins or GitLab CI.

3. **Difference between Dockerfile and Docker Compose.**
   - **Dockerfile**: Defines how to build a single container.
   - **Docker Compose**: Defines and manages multiple containers.

4. **Contents of `service.yaml` and `deployment.yaml`:**
   - **service.yaml**:
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: my-service
     spec:
       selector:
         app: my-app
       ports:
         - protocol: TCP
           port: 80
           targetPort: 8080
     ```
   - **deployment.yaml**:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: my-deployment
     spec:
       replicas: 3
       selector:
         matchLabels:
           app: my-app
       template:
         metadata:
           labels:
             app: my-app
         spec:
           containers:
             - name: my-container
               image: nginx
     ```

5. **What is a multistage Dockerfile?**
   - A Dockerfile that uses multiple stages to optimize the build process, reducing image size.

6. **How to secure a Kubernetes cluster?**
   - Enable **RBAC**.
   - Use **network policies**.
   - Enable **audit logging**.
   - Regularly patch and update Kubernetes.

7. **How do pods access secrets?**
   - Secrets can be mounted as **environment variables** or as **volumes** in pods.

8. **What is a headless service and sidecar container?**
   - **Headless Service**: A service without a ClusterIP, allowing direct pod-to-pod communication.
   - **Sidecar Container**: A container that runs alongside the main container to provide auxiliary functions (e.g., logging, monitoring).

9. **How to upgrade a Kubernetes cluster (step-by-step)?**
   - Backup **etcd** data.
   - Upgrade **control plane nodes**.
   - Upgrade **worker nodes**.
   - Upgrade **kubectl** and apply updated manifests.
