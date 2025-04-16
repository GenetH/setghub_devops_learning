# Orchestrating_Containers_across_Multiple_Virtual_Servers_with_Kubernetes

Containers are the most lightweight and easily transferrable workloads, they start faster than VMs, consume less space and memory, and they perfectly fit to accommodate microservice architecture. It means that the number of containers will be usually significantly higher than the number of VMs, and with the number comes the complexity of managing containers.

To manage a fleet of containers, you can use different software solutions, but the most widely used one is Kubernetes (also referred to as K8s). It was originally developed by Google and now is being maintained by the Cloud Native Computing Foundation (CNCF).

Why has it become so popular? It is a powerful Swiss-army knife when it comes to managing multiple containers in production-grade solutions.

For now, we will not list down all the features and benefits of Kubernetes, but you can watch [this overview video](#) to get to know more. You certainly must have this tool in your DevOps arsenal and know when and how to use it for production-grade deployments.

This project is the first of a series Kubernetes-related practice projects, so get ready to become a professional containers' fleet "pilot"*.

*Kubernetes (κubernetes, Greek for "helmsman" or "pilot" or "governor", and the etymological root of cybernetics)

**Why migrate from Docker Compose to K8s**

In the previous project, you successfully deployed your Docker containers using **Docker Compose**, which is a great tool that helps avoid the execution of multiple CLI commands by preparing a declarative configuration file. It is handy when you deploy one or a few containers, but in most cases, it does not fit for production deployments.

Because of the many limitations that Docker Compose has, it is very important for us to consider migrating our solution to more advanced technology. The most common alternatives to Compose, amongst a few others, are **Docker Swarm** and **Kubernetes**.

**What is wrong with Docker Compose?**

It is important to understand that, DevOps is about "Culture" and NOT "Tools." Therefore, it is not correct to say that one tool is better than another; different organizations have different needs, and a good tool for one team may be bad for another just because their needs are not the same. In some teams, Docker Compose fits their needs perfectly, despite the perceived limitations. The major limitation of Docker Compose is that it can only be used to run workloads on a single computer host. Now, that is an obvious limitation because if our **Tooling Application** and its **MySQL Database** are all running on a single VM, like we did in Project 20, then this host is considered as a **SPOF** (i.e. - Single Point of Failure).

So, could we say there is something wrong with Docker Compose? Not exactly, as a matter of fact, it is being used a lot in the industry. It fits well into some use cases that require speedy development and Proof of Concepts. As you will soon see, **Kubernetes** is a lot more complex technology, and it may be an overkill for some use cases.

**Container Orchestration with Kubernetes**  
**What is Container Orchestration?**

Two important things to remember about Docker containers are:

1. Unlike virtual machines, they are not designed to run for a very long time. By design, Docker containers are ephemeral. By "ephemeral," it means that a container can be stopped and destroyed, and a new one can be built from the same Docker image and put in place with an absolute minimum set-up and configuration requirement.
   
2. To ensure that container workloads are highly scalable, they must be configured to run across multiple compute nodes. A compute node can be a physical server or a virtual machine with Docker engine installed to run your containers.

If we had two compute nodes to run our containers, let us consider the following scenarios:

1. Given the two points mentioned above, if containers are configured to run across 2 computer nodes, and a particular container, running on Node 1 dies, how will it know that it can spin up again on Node 2?
   
2. Let us imagine that the Tooling website container is running on Node 1, and the MySQL container is running on Node 2, how will both containers be able to communicate with each other?  
   Remember in Project 20, we had to create a custom network on the same host and ensure that they can communicate through that network. But in the case of 2 separate hosts, this is natively not possible.

**Container orchestration** is a concept that allows to address these two scenarios, it provides automation of all the aspects of coordinating and managing containers. Container orchestration is focused on managing the life cycle of containers and their dynamic environments.

It is about automating the entire lifecycle of containers running across multiple nodes:

- Configuring and scheduling of containers on nodes
- Ensuring the availability of containers, even when they die
- Scaling of containers to equally balance application workloads across infrastructure
- Allocation of resources between containers
- Load balancing, traffic routing, and service discovery of containers
- Health monitoring of containers
- Securing the interactions between containers.

**Kubernetes** is a tool designed to do Container Orchestration and it does its job very well when correctly configured.

As mentioned earlier, there are other alternatives to Docker Compose. But, throughout the entire PBL program, we will not focus on **Docker Swarm**. We will rather spend more time with **Kubernetes**. Part of the reason for this is because Kubernetes has more functionalities and is widely in use in the industry.

To know when to choose between **Docker Swarm** and **Kubernetes**, [Here is an interesting article](#) to read with some very enlightening stats.

**Kubernetes architecture**

Kubernetes is not a single package application that you can install with one command, it is comprised of several components, some of them can be deployed as services, some can be also deployed as separate containers.

Let us take a look at Kubernetes architecture diagram below:

![AWS Solution](./self_study/images/kub.png)

Read about every component in the [official documentation](https://kubernetes.io/docs/).

Make sure you understand the role of each component on the diagram above, without this understanding it will be extremely difficult for you to install and operate a K8s cluster, especially when it comes to troubleshooting and maintenance.

As an IT professional in general, you shall be comfortable using official documentation for tools you use, in case of Kubernetes - it has a very well structured and comprehensive documentation portal with multiple configuration code snippets. We strongly encourage you to add it to your bookmarks and refer to it every time you have a K8s-related question.

**"Kubernetes From-Ground-Up"  
K8s installation options**

__DISCLAIMER:__ The following setup of Kubernetes should be used for learning purpose only, and not to be considered for production. This is because setting up a K8s cluster for production use has a lot more moving parts, especially when it comes to planning the nodes, and securing the cluster. The purpose of __`K8s From-Ground-Up`__ is to get you much closer to the different components as shown in the architecture diagram and relate with what you have been learning about Kubernetes.

## Tools to be used and expected result of the Project 20

- VM: AWS EC2
- OS: Ubuntu 20.04 lts+
- Docker Engine
- kubectl console utility
- cfssl and cfssljson utilities
- Kubernetes cluster

We will create `6 EC2 Instances`, and in the end, we will have the following parts of the cluster properly configured:

- Three Kubernetes Master
- Three Kubernetes Worker Nodes
- Configured SSL/TLS certificates for Kubernetes components to communicate securely
- Configured Node Network
- Configured Pod Network

**Step 0: Install client tools before bootstrapping the cluster.**

First, you will need some client tools installed and configurations made on your client workstation:

- **awscli** – is a unified tool to manage your AWS services.
- **kubectl** – this command line utility will be your main control tool to manage your K8s cluster. You will use this tool so many times, so you will be able to type ‘kubectl’ on your keyboard with a speed of light. You can always make a shortcut (alias) to just one character 'k'. Also, add this extremely useful official kubectl Cheat Sheet to your bookmarks, it has examples of the most used ‘kubectl’ commands.
- **cfssl** – an open-source toolkit for everything TLS/SSL from Cloudflare.
- **cfssljson** – a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.


**Install and configure AWS CLI**

Configure AWS CLI to access all AWS services used, for this we need to have a user with `programmatic access keys` configured in AWS Identity and Access Management (IAM):

![AWS Solution](./self_study/images/a.png)

On your local workstation, download and install the [latest version of AWS CLI](https://aws.amazon.com/cli/).

To configure your AWS CLI – run your shell (or cmd if using Windows) and run:

```bash
$ aws configure --profile %your_username%
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

Test your AWS CLI by running:

```bash
aws ec2 describe-vpcs
```
![AWS Solution](./self_study/images/b.png)

**Install kubectl**

Kubernetes cluster has a Web API that can receive HTTP/HTTPS requests, but it is quite cumbersome to `curl` an API each and every time you need to send some command, so `kubectl` command tool was developed to ease a K8s administrator's life.

With this tool you can easily interact with Kubernetes to deploy applications, inspect and manage cluster resources, view logs, and perform many more administrative operations.

**Linux Or Windows using Gitbash or similar tool**

1. **Download the binary:**
   ```bash
   wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
   ```

2. **Make it executable:**
   ```bash
   chmod +x kubectl
   ```

3. **Move to the Bin directory:**
   ```bash
   sudo mv kubectl /usr/local/bin/
   ```

4. **Verify that kubectl version 1.21.0 or higher is installed:**
   ```bash
   kubectl version --client
   ```
**Output:**
```bash
Client Version: version.Info{Major:"1", Minor:"20+", GitVersion:"v1.20.4-dirty", GitCommit:"e87da0bd6e03ec3fea7933c4b5263d151aadf07c", GitTreeState:"dirty", BuildDate:"2021-03-15T10:03:32Z", GoVersion:"go1.16.2", Compiler:"gc", Platform:"darwin/amd64"}
```
![AWS Solution](./self_study/images/c.png)

**Install CFSSL and CFSSLJSON**

`cfssl` is an open-source tool by Cloudflare used to set up a Public Key Infrastructure (PKI) for generating, signing, and bundling TLS certificates. In previous projects, you have experienced the use of Let’s Encrypt for a similar use case. Here, `cfssl` will be configured as a Certificate Authority, which will issue the certificates required to spin up a Kubernetes cluster.

Download, install and verify the successful installation of `cfssl` and `cfssljson`:

**Linux or Windows using Gitbash or similar tool:**

```bash
wget -q --show-progress --https-only --timestamping \
https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
```

```bash
chmod +x cfssl cfssljson
```

```bash
sudo mv cfssl cfssljson /usr/local/bin/
```
![AWS Solution](./self_study/images/d.png)

**AWS Cloud resources for Kubernetes Cluster**

As we already know, we need some machines to run the control plane and the worker nodes. In this section, you will provision EC2 Instances required to run your K8s cluster. You can use Terraform for this. But it is highly recommended to start out first with manual provisioning using `awscli` and have thorough knowledge about the whole setup. After that, you can destroy the entire project and start all over again using Terraform. This manual approach will solidify your skills and give you the opportunity to face more challenges.

### Step 1 - Configure Network Infrastructure

**Virtual Private Cloud - VPC**

1. Create a directory named `k8s-cluster-from-ground-up`
2. Create a VPC and store the ID as a variable:

```bash
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 172.31.0.0/16 \
  --output text --query 'Vpc.VpcId' \
)
```
3. Tag the VPC so that it is named:

```bash
NAME=k8s-cluster-from-ground-up

aws ec2 create-tags \
  --resources ${VPC_ID} \
  --tags Key=Name,Value=${NAME}
```

**Domain Name System - DNS**

4. Enable DNS support for your VPC:

```bash
aws ec2 modify-vpc-attribute \
  --vpc-id ${VPC_ID} \
  --enable-dns-support '{"Value": true}'
```
![AWS Solution](./self_study/images/e.png)

5. Enable DNS support for hostnames:

```bash
aws ec2 modify-vpc-attribute \
  --vpc-id ${VPC_ID} \
  --enable-dns-hostnames '{"Value": true}'
```
![AWS Solution](./self_study/images/g.png)

![AWS Solution](./self_study/images/k.png)

**AWS Region**

6. Set the required region

```bash
AWS_REGION=eu-central-1
```

**Subnet**

7. Create the Subnet:

```bash
SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 172.31.0.0/24 \
  --output text --query 'Subnet.SubnetId')

aws ec2 create-tags \
  --resources ${SUBNET_ID} \
  --tags Key=Name,Value=${NAME}
```
![AWS Solution](./self_study/images/s.png)

**Internet Gateway - IGW**

8. Create the Internet Gateway and attach it to the VPC:

```bash
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
  --output text --query 'InternetGateway.InternetGatewayId')
aws ec2 create-tags \
  --resources ${INTERNET_GATEWAY_ID} \
  --tags Key=Name,Value=${NAME}
aws ec2 attach-internet-gateway \
  --internet-gateway-id ${INTERNET_GATEWAY_ID} \
  --vpc-id ${VPC_ID}
```
![AWS Solution](./self_study/images/h.png)

**Route tables**

9. Create route tables, associate the route table to the subnet, and create a route to allow external traffic to the Internet through the Internet Gateway:

```bash
ROUTE_TABLE_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --output text --query 'RouteTable.RouteTableId')
aws ec2 create-tags \
  --resources ${ROUTE_TABLE_ID} \
  --tags Key=Name,Value=${NAME}
aws ec2 associate-route-table \
  --route-table-id ${ROUTE_TABLE_ID} \
  --subnet-id ${SUBNET_ID}
aws ec2 create-route \
  --route-table-id ${ROUTE_TABLE_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${INTERNET_GATEWAY_ID}
```

**Output:**

```json
{
   "AssociationId": "rtbassoc-07a8877e92504def7",
   "AssociationState": {
      "State": "associated"
   }
}
{
   "Return": true
}
```
![AWS Solution](./self_study/images/sj.png)
![AWS Solution](./self_study/images/sk.png)

**Security Groups**

10. Configure security groups

```bash
# Create the security group and store its ID in a variable
SECURITY_GROUP_ID=$(aws ec2 create-security-group \
  --group-name ${NAME} \
  --description "Kubernetes cluster security group" \
  --vpc-id ${VPC_ID} \
  --output text --query 'GroupId')

# Create the NAME tag for the security group
aws ec2 create-tags \
  --resources ${SECURITY_GROUP_ID} \
  --tags Key=Name,Value=${NAME}

# Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

# # Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'

# Create inbound traffic to allow connections to the Kubernetes API Server listening on port 6443
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 6443 \
  --cidr 0.0.0.0/0

# Create Inbound traffic for SSH from anywhere (Do not do this in production. Limit access ONLY to IPs or CIDR that MUST connect)
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Create ICMP ingress for all types
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol icmp \
  --port -1 \
  --cidr 0.0.0.0/0
```

![AWS Solution](./self_study/images/js.png)
![AWS Solution](./self_study/images/ff.png)

**Network Load Balancer**

11. Create a network Load Balancer

```bash
LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer \
  --name ${NAME} \
  --subnets ${SUBNET_ID} \
  --scheme internet-facing \
  --type network \
  --output text --query 'LoadBalancers[].LoadBalancerArn')
```
![AWS Solution](./self_study/images/nk.png)
![AWS Solution](./self_study/images/nl.png)

**Target Group**

12. Create a target group: (For now it will be unhealthy because there are no real targets yet.)

```bash
TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
  --name ${NAME} \
  --protocol TCP \
  --port 6443 \
  --vpc-id ${VPC_ID} \
  --target-type ip \
  --output text --query 'TargetGroups[].TargetGroupArn')
```
![AWS Solution](./self_study/images/km.png)
![AWS Solution](./self_study/images/hell.png)

13. Register targets: (Just like above, no real targets. You will just put the IP addresses so that, when the nodes become available, they will be used as targets.)
```bash
aws elbv2 register-targets \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --targets Id=172.31.0.1{0,1,2}
```
![AWS Solution](./self_study/images/boo.png)
![AWS Solution](./self_study/images/tar.png)

14. Create a listener to listen for requests and forward to the target nodes on TCP port 6443
```bash
aws elbv2 create-listener \
  --load-balancer-arn ${LOAD_BALANCER_ARN} \
  --protocol TCP \
  --port 6443 \
  --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} \
  --output text --query 'Listeners[].ListenerArn'
```
![AWS Solution](./self_study/images/dar.png)
![AWS Solution](./self_study/images/tarr.png)

**K8s Public Address**
15. Get the Kubernetes Public address
```bash
KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
  --load-balancer-arns ${LOAD_BALANCER_ARN} \
  --output text --query 'LoadBalancers[].DNSName')
```
![AWS Solution](./self_study/images/kube.png)

### Step 2 -  Create Compute Resources

**AMI**
1. Get an image to create EC2 instances:
```bash
IMAGE_ID=$(aws ec2 describe-images --owners 099720109477 \
  --filters \
  'Name=root-device-type,Values=ebs' \
  'Name=architecture,Values=x86_64' \
  'Name=name,Values=ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*' \
  | jq -r '.Images|sort_by(.Name)[-1]|.ImageId')
```
![AWS Solution](./self_study/images/ima.png)

**SSH key-pair**
2. Create SSH Key-Pair
```bash
mkdir -p ssh

aws ec2 create-key-pair \
  --key-name ${NAME} \
  --output text --query 'KeyMaterial' \
  > ssh/${NAME}.id_rsa
chmod 600 ssh/${NAME}.id_rsa
```
![AWS Solution](./self_study/images/yu.png)

**SSH key-pair**
3. Create 3 Master nodes: Note - Using t2.micro instead of t2.small as t2.micro is covered by AWS free tier
```bash