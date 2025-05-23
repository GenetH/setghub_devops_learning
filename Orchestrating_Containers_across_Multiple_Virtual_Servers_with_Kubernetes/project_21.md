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


### Step 0: Install client tools before bootstrapping the cluster.

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

**EC2 Instances for Controle Plane (Master Nodes)**
3. Create 3 Master nodes: Note - Using t2.micro instead of t2.small as t2.micro is covered by AWS free tier
```bash
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name ${NAME} \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.1${i} \
    --user-data "name=master-${i}" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=${NAME}-master-${i}"
done
```
![AWS Solution](./self_study/images/dog.png)
![AWS Solution](./self_study/images/ins.png)

**EC2 Instances for Worker Nodes**
4. Create 3 worker nodes:
```bash
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name ${NAME} \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.2${i} \
    --user-data "name=worker-${i}|pod-cidr=172.20.${i}.0/24" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=${NAME}-worker-${i}"
done
```
![AWS Solution](./self_study/images/rest.png)
![AWS Solution](./self_study/images/goo.png)

### Step 3 - Prepare The Self-Signed Certificate Authority And Generate TLS Certificates

The following components running on the Master node will require TLS certificates.

- kube-controller-manager
- kube-scheduler
- etcd
- kube-apiserver
- kubelet
- kube-proxy

The following components running on the Worker nodes will require TLS certificates.

- kubelet
- kube-proxy

Therefore, you will provision a `PKI` Infrastructure using cfssl which will have a Certificate Authority. The CA will then generate certificates for all the individual components.

**Self-Signed Root Certificate Authority (CA)**

Here, you will provision a `CA` that will be used to sign additional TLS certificates.

Create a directory and `cd` into it:

```bash
mkdir ca-authority && cd ca-authority
```
Generate the CA configuration file, Root Certificate, and Private key:

```bash
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```
The file defines the following:

```css
CN – Common name for the authority

algo – the algorithm used for the certificates

size – algorithm size in bits

C – Country

L – Locality (city)

ST – State or province

O – Organization

OU – Organizational Unit
```
__Output:__

```bash
2021/05/16 20:18:44 [INFO] generating a new CA key and certificate from CSR
2021/05/16 20:18:44 [INFO] generate received request
2021/05/16 20:18:44 [INFO] received CSR
2021/05/16 20:18:44 [INFO] generating key: rsa-2048
2021/05/16 20:18:44 [INFO] encoded CSR
2021/05/16 20:18:44 [INFO] signed certificate with serial number 478642753175858256977534824638605235819766817855
```
![AWS Solution](./self_study/images/baba.png)
![AWS Solution](./self_study/images/bal.png)

List the directory to see the created files

```bash
ls -ltr

-rw-r--r--  1 dare  dare   232 16 May 20:18 ca-config.json
-rw-r--r--  1 dare  dare   207 16 May 20:18 ca-csr.json
-rw-r--r--  1 dare  dare  1306 16 May 20:18 ca.pem
-rw-------  1 dare  dare  1679 16 May 20:18 ca-key.pem
-rw-r--r--  1 dare  dare  1001 16 May 20:18 ca.csr
```
![AWS Solution](./self_study/images/ls.png)

The 3 important files here are:

- __ca.pem__ - The Root Certificate
- __ca-key.pem__ - The Private Key
- __ca.csr__ - The Certificate Signing Request

### Generating TLS Certificates For Client and Server

You will need to provision Client/Server certificates for all the components. It is a `MUST` to have encrypted communication within the cluster. Therefore, the `server` here are the master nodes running the `api-server` component. While the `client` is every other component that needs to communicate with the api-server.

Now we have a certificate for the Root CA, we can then begin to request more certificates which the different Kubernetes components, i.e. clients and server, will use to have encrypted communication.

Remember, the clients here refer to every other component that will communicate with the api-server. These are:

- kube-controller-manager
- kube-scheduler
- etcd
- kubelet
- kube-proxy
- Kubernetes Admin User

### Let us begin with the Kubernetes API-Server Certificate and Private Key

The certificate for the Api-server must have IP addresses, DNS names, and a Load Balancer address included. Otherwise, you will have a lot of difficulties connecting to the `api-server`.

1. Generate the __Certificate Signing Request (CSR)__, __Private Key__ and the __Certificate__ for the Kubernetes Master Nodes.

```bash
{
cat > master-kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
   "hosts": [
   "127.0.0.1",
   "172.31.0.10",
   "172.31.0.11",
   "172.31.0.12",
   "ip-172-31-0-10",
   "ip-172-31-0-11",
   "ip-172-31-0-12",
   "ip-172-31-0-10.${AWS_REGION}.compute.internal",
   "ip-172-31-0-11.${AWS_REGION}.compute.internal",
   "ip-172-31-0-12.${AWS_REGION}.compute.internal",
   "${KUBERNETES_PUBLIC_ADDRESS}",
   "kubernetes",
   "kubernetes.default",
   "kubernetes.default.svc",
   "kubernetes.default.svc.cluster",
   "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  master-kubernetes-csr.json | cfssljson -bare master-kubernetes
}
```
![AWS Solution](./self_study/images/all.png)
![AWS Solution](./self_study/images/alll.png)

### Creating the other certificates: for the following Kubernetes components:

- Scheduler Client Certificate
- Kube Proxy Client Certificate
- Controller Manager Client Certificate
- Kubelet Client Certificates
- K8s admin user Client Certificate

2. __`kube-scheduler` Client Certificate and Private Key__

```bash
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-scheduler",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```
![AWS Solution](./self_study/images/war.png)

_If you see any warning message, it is safe to ignore it._

3. __`kube-proxy` Client Certificate and Private Key__

```bash
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:node-proxier",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```
![AWS Solution](./self_study/images/sir.png)

4. __`kube-controller-manager` Client Certificate and Private Key__

```bash
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-controller-manager",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```
![AWS Solution](./self_study/images/fot.png)

5. __`kubelet` Client Certificate and Private Key__

Similar to how you configured the `api-server's` certificate, Kubernetes requires that the hostname of each worker node is included in the client certificate.

Also, Kubernetes uses a special-purpose authorization mode called [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/), that specifically authorizes API requests made by `kubelet` services. In order to be authorized by the Node Authorizer, `kubelets` must use a credential that identifies them as being in the `system:nodes` __group__, with a username of `system:node:<nodeName>`. Notice the `"CN": "system:node:${instance_hostname}"`, in the below code.

Therefore, the certificate to be created must comply to these requirements. In the below example, there are 3 worker nodes, hence we will use bash to loop through a list of the worker nodes' hostnames, and based on each index, the respective Certificate Signing Request (CSR), private key and client certificates will be generated.

```bash
for i in 0 1 2; do
  instance="${NAME}-worker-${i}"
  instance_hostname="ip-172-31-0-2${i}"
  cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance_hostname}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:nodes",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')

  internal_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PrivateIpAddress')

  cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=ca-config.json \
    -hostname=${instance_hostname},${external_ip},${internal_ip} \
    -profile=kubernetes \
    ${NAME}-worker-${i}-csr.json | cfssljson -bare ${NAME}-worker-${i}
done
```
![AWS Solution](./self_study/images/po.png)

6. __Finally, `kubernetes admin user's` Client Certificate and Private Key__

```bash
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:masters",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}
```
![AWS Solution](./self_study/images/fii.png)

7. __Actually, we are not done yet!__

There is one more pair of certificate and private key we need to generate. That is for the __Token Controller:__ a part of the Kubernetes Controller Manager `kube-controller-manager` responsible for generating and signing service account tokens which are used by _pods_ or other _resources_ to establish connectivity to the `api-serve`r`. Read more about Service Accounts from the [official documentation](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/).

Alright, let us quickly create the last set of files, and we are done with PKIs

```bash
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "STEGHUB DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
}
```
![AWS Solution](./self_study/images/7.png)

### Step 4 - Distributing the Client and Server Certificates

Now it is time to start sending all the client and server certificates to their respective instances.

Let us begin with the __worker nodes:__

Copy these files securely to the worker nodes using `scp` utility

- Root CA certificate - ca.pem
- X509 Certificate for each worker node
- Private Key of the certificate for each worker node

```bash
for i in 0 1 2; do
  instance="${NAME}-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
    ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
```
OUTPUT:
```bash
ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
ca.pem                                                                                                                                                                             100% 1350    48.2KB/s   00:00
k8s-cluster-from-ground-up-worker-0-key.pem                                                                                                                                        100% 1675    52.5KB/s   00:00
k8s-cluster-from-ground-up-worker-0.pem                                                                                                                                            100% 1594    48.9KB/s   00:00
ca.pem                                                                                                                                                                             100% 1350    35.9KB/s   00:00
k8s-cluster-from-ground-up-worker-1-key.pem                                                                                                                                        100% 1675    41.6KB/s   00:00
k8s-cluster-from-ground-up-worker-1.pem                                                                                                                                            100% 1594    44.0KB/s   00:00
ca.pem                                                                                                                                                                             100% 1350    44.7KB/s   00:00
k8s-cluster-from-ground-up-worker-2-key.pem                                                                                                                                        100% 1679    49.2KB/s   00:00
k8s-cluster-from-ground-up-worker-2.pem
```
![AWS Solution](./self_study/images/cant.png)

__Master or Controller node:__ - Note that only the `api-server` related files will be sent over to the master nodes.

```bash
for i in 0 1 2; do
instance="${NAME}-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem \
    master-kubernetes.pem master-kubernetes-key.pem ubuntu@${external_ip}:~/;
done
```
__Output:__
```bash
ca.pem                                                                                                                                                                             100% 1350     8.4KB/s   00:00
ca-key.pem                                                                                                                                                                         100% 1675    44.7KB/s   00:00
service-account-key.pem                                                                                                                                                            100% 1675    45.3KB/s   00:00
service-account.pem                                                                                                                                                                100% 1440    42.0KB/s   00:00
master-kubernetes.pem                                                                                                                                                              100% 1956    58.5KB/s   00:00
master-kubernetes-key.pem                                                                                                                                                          100% 1671    47.5KB/s   00:00
ca.pem                                                                                                                                                                             100% 1350    42.9KB/s   00:00
ca-key.pem                                                                                                                                                                         100% 1675    46.3KB/s   00:00
service-account-key.pem                                                                                                                                                            100% 1675    44.1KB/s   00:00
service-account.pem                                                                                                                                                                100% 1440    46.9KB/s   00:00
master-kubernetes.pem                                                                                                                                                              100% 1956    54.6KB/s   00:00
master-kubernetes-key.pem                                                                                                                                                          100% 1671    48.7KB/s   00:00
ca.pem                                                                                                                                                                             100% 1350    41.8KB/s   00:00
ca-key.pem                                                                                                                                                                         100% 1675    45.4KB/s   00:00
service-account-key.pem                                                                                                                                                            100% 1675    52.5KB/s   00:00
service-account.pem                                                                                                                                                                100% 1440    45.6KB/s   00:00
master-kubernetes.pem                                                                                                                                                              100% 1956    48.9KB/s   00:00
master-kubernetes-key.pem
```
![AWS Solution](./self_study/images/yes.png)

The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet client` certificates will be used to generate client authentication configuration files later.

### Step 4 - Use `kubectl` to Generate Kubernetes Configuration Files for Authentication

All the work you are doing right now is ensuring that you do not face any difficulties by the time the Kubernetes cluster is up and running. In this step, you will create some files known as `kubeconfig`, which enables Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

You will need a client tool called `kubectl` to do this. And, by the way, most of your time with Kubernetes will be spent using kubectl commands.

Now it's time to generate kubeconfig files for the `kubelet`, `controller manager`, `kube-proxy`, and `scheduler client`s and then the `admin user`.

First, let us create a few environment variables for reuse by multiple commands.

```bash
KUBERNETES_API_SERVER_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')
```
![AWS Solution](./self_study/images/duu.png)

1. Generate the `kubelet` kubeconfig file

For each of the nodes running the kubelet component, it is very important that the client certificate configured for that node is used to generate the kubeconfig. This is because each certificate has the node's DNS name or IP Address configured at the time the certificate was generated. It will also ensure that the appropriate authorization is applied to that node through the [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/)

Below command must be run in the directory where all the certificates were generated.

```bash
for i in 0 1 2; do

instance="${NAME}-worker-${i}"
instance_hostname="ip-172-31-0-2${i}"

# Set the kubernetes cluster in the kubeconfig file
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://$KUBERNETES_API_SERVER_ADDRESS:6443 \
    --kubeconfig=${instance}.kubeconfig

# Set the cluster credentials in the kubeconfig file
  kubectl config set-credentials system:node:${instance_hostname} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

# Set the context in the kubeconfig file
  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:node:${instance_hostname} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```
__Output:__
```bash
Cluster "k8s-cluster-from-ground-up" set.

User "system:node:ip-172-31-0-20.eu-central-1.compute.internal" set.

Context "default" created.

Switched to context "default".

Cluster "k8s-cluster-from-ground-up" set.

User "system:node:ip-172-31-0-21.eu-central-1.compute.internal" set.

Context "default" created.

Switched to context "default".

Cluster "k8s-cluster-from-ground-up" set.

User "system:node:ip-172-31-0-22.eu-central-1.compute.internal" set.

Context "default" created.

Switched to context "default".
```
![AWS Solution](./self_study/images/f1.png)

List the output

```bash
ls -ltr *.kubeconfig
```
OUTPUT:
```bash
-rw-------  1 dare  staff  6602 22 Jun 20:40 k8s-cluster-from-ground-up-worker-0.kubeconfig
-rw-------  1 dare  staff  6602 22 Jun 20:40 k8s-cluster-from-ground-up-worker-1.kubeconfig
-rw-------  1 dare  staff  6606 22 Jun 20:40 k8s-cluster-from-ground-up-worker-2.kubeconfig
```
![AWS Solution](./self_study/images/f2.png)

Open up the kubeconfig files generated and review the 3 different sections that have been configured:

- Cluster
- Credentials
- And Kube Context

Kubeconfig file is used to organize information about clusters, users, namespaces and authentication mechanisms. By default, `kubectl` looks for a file named `config` in the `$HOME/.kube` directory. You can specify other kubeconfig files by setting the `KUBECONFIG` environment variable or by setting the `--kubeconfig` flag. To get to know more how to create your own kubeconfig files - read this [documentation](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/).

__Context__ part of `kubeconfig` file defines three main parameters: cluster, namespace and user. You can save several different contexts with any convenient names and switch between them when needed.

```bash
kubectl config use-context %context-name%
```
2. Generate the `kube-proxy` kubeconfig

```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```
![AWS Solution](./self_study/images/f3.png)

3. Generate the __Kube-Controller-Manager__ kubeconfig

Notice that the `--server` is set to use `127.0.0.1`. This is because, this component runs on the API-Server so there is no point routing through the Load Balancer.

```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```
![AWS Solution](./self_study/images/f4.png)

4. Generating the __Kube-Scheduler__ Kubeconfig

```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```
![AWS Solution](./self_study/images/f5.png)


5. Finally, generate the `kubeconfig` file for the `admin user`

```bash
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```
![AWS Solution](./self_study/images/f6.png)

__TASK: Distribute the files to their respective servers, using `scp` and a `for loop` like we have done previously. This is a test to validate that you understand which component must go to which node.__

__For Worker Nodes__

Copy the config files for kublet and kube-proxy to the worker nodes.

```bash
for i in 0 1 2; do
   instance="${NAME}-worker-${i}"
   external_ip=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=${instance}" \
     --output text --query 'Reservations[].Instances[].PublicIpAddress')
    scp -i ../ssh/${NAME}.id_rsa \
    ${instance}.kubeconfig kube-proxy.kubeconfig ubuntu@${external_ip}:~/
done
```
![AWS Solution](./self_study/images/f7.png)
![AWS Solution](./self_study/images/f8.png)

__For Master Nodes__

Copy the kube-controller-manager, kube-scheduler kubeconfig files and admin kubeconfig file to each master node

```bash
for i in 0 1 2; do
instance="${NAME}-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
  kube-scheduler.kubeconfig kube-controller-manager.kubeconfig admin.kubeconfig ubuntu@${external_ip}:~/;
done
```
![AWS Solution](./self_study/images/9.png)

### Step 5 - Prepare the etcd database for encryption at rest.

Kubernetes uses [etcd](https://etcd.io/) (A distributed key value store) to store variety of data which includes the cluster state, application configurations, and secrets. By default, the data that is being persisted to the disk is not encrypted. Any attacker that is able to gain access to this database can exploit the cluster since the data is stored in plain text. Hence, it is a security risk for Kubernetes that needs to be addressed.

To mitigate this risk, we must prepare to encrypt `etcd` at rest. "At rest" means data that is stored and persists on a disk. Anytime you hear `in-flight` or `in transit` refers to data that is being transferred over the network. "In-flight" encryption is done through `TLS`.

__Generate the encryption key and encode it using `base64`__

```bash
ETCD_ENCRYPTION_KEY=$(head -c 64 /dev/urandom | base64)
```
See the output that will be generated when called. Yours will be a different random string.

```bash
echo $ETCD_ENCRYPTION_KEY
```
OUTPUT:
```
OuxSvV5XUQVid4fNNbeyFEDTUPr1yozZPQ+E6Eqj80m1FSVDB6jOHt9miD/7kMdJIvVshlMgxY80wFajlqItug===$
```
![AWS Solution](./self_study/images/f9.png)

__Create an `encryption-config.yaml` file as [documented officially by kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration)__

```bash
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ETCD_ENCRYPTION_KEY}
      - identity: {}
EOF
```
![AWS Solution](./self_study/images/f10.png)

Send the encryption file to the __Controller__ nodes using `scp` and a `for loop`.

```bash
for i in 0 1 2; do
instance="${NAME}-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
  encryption-config.yaml ubuntu@${external_ip}:~/;
done
```
![AWS Solution](./self_study/images/f11.png)

### Bootstrap `etcd` cluster

__TIPS:__ Use a terminal multi-plexer like multi-tabbed putty or tmux to work with multiple terminal sessions simultaneously. It will make your life easier, especially when you need to work on multiple nodes and run the same command across all nodes. Imagine repeating the same commands on 10 different nodes, and you don not intend to start automating with a configuration management tool like Ansible `yet`.

The primary purpose of the `etcd` component is to store the state of the cluster. This is because Kubernetes itself is stateless. Therefore, all its stateful data will persist in `etcd`. Since Kubernetes is a distributed system - it needs a distributed storage to keep persistent data in it. `etcd` is a highly-available key value store that fits the purpose. All K8s cluster configurations are stored in a form of key value pairs in `etcd`, it also stores the __actual__ and __desired__ states of the cluster. `etcd` cluster is intelligent enough to watch for changes made on one instance and almost instantly replicate those changes to the rest of the instances, so all of them will be always reconciled.

__NOTE:__ Use tools like tmux to make it easy to run commands on multiple terminal screens at once.


1. __SSH into the controller server__

- __Master-1__

```bash
export NAME=k8s-cluster-from-ground-up

master_1_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-master-0" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_1_ip}
```

- __Master-2__

```bash
 export NAME=k8s-cluster-from-ground-up

master_2_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-master-1" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_2_ip}
```

- __Master-3__

```bash
export NAME=k8s-cluster-from-ground-up

master_3_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-master-2" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_3_ip}
```
You should have a similar pane like below. You should be able to see all the files that have been sent to the nodes.

![AWS Solution](./self_study/images/f21.png)

![AWS Solution](./self_study/images/f20.png)

2. __Download and install `etcd`__

```bash
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.15/etcd-v3.4.15-linux-amd64.tar.gz"
```
3. Extract and install the etcd server and the etcdctl command line utility:

```bash
{
  tar -xvf etcd-v3.4.15-linux-amd64.tar.gz
  sudo mv etcd-v3.4.15-linux-amd64/etcd* /usr/local/bin/
}
```

4. Configure the `etcd` server

```bash
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem master-kubernetes-key.pem master-kubernetes.pem /etc/etcd/
}
```
![AWS Solution](./self_study/images/f23.png)

5. The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:

```bash
# IMDSv2 requires session tokens for accessing metadata
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

export INTERNAL_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/local-ipv4)

echo $INTERNAL_IP
```
![AWS Solution](./self_study/images/f18.png)

6. Each etcd member must have a unique name within an etcd cluster. Set the etcd name to node Private IP address so it will uniquely identify the machine:

```bash
ETCD_NAME=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo ${ETCD_NAME}
```
![AWS Solution](./self_study/images/f17.png)

7. Create the `etcd.service` systemd unit file:

The flags are well documented [here](https://www.bookstack.cn/read/etcd-3.2.17-en/717bafd59fa87192.md)

```bash
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://172.31.0.10:2380,master-1=https://172.31.0.11:2380,master-2=https://172.31.0.12:2380 \\
  --cert-file=/etc/etcd/master-kubernetes.pem \\
  --key-file=/etc/etcd/master-kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/master-kubernetes.pem \\
  --peer-key-file=/etc/etcd/master-kubernetes-key.pem \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
![AWS Solution](./self_study/images/m1.png)

8. Start and enable the `etcd` Server

```bash
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

9. Verify the etcd installation

```bash
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/master-kubernetes.pem \
  --key=/etc/etcd/master-kubernetes-key.pem
```
Output:
```bash
6709c481b5234095, started, master-0, https://172.31.0.10:2380, https://172.31.0.10:2379, false
ade74a4f39c39f33, started, master-1, https://172.31.0.11:2380, https://172.31.0.11:2379, false
ed33b44c0b153ee3, started, master-2, https://172.31.0.12:2380, https://172.31.0.12:2379, false
```
![AWS Solution](./self_study/images/m2.png)

```bash
systemctl status etcd
```
![AWS Solution](./self_study/images/m3.png)

### Bootstrap the `Control Plane`

In this section, you will configure the components for the control plane on the master/controller nodes.

1. Create the Kubernetes configuration directory:

```bash
sudo mkdir -p /etc/kubernetes/config
```

2. Download the official Kubernetes release binaries:

```bash
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl"
```
![AWS Solution](./self_study/images/m5.png)

3. Install the Kubernetes binaries:

```bash
{
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

4. Configure the Kubernetes API Server

```bash
{
  sudo mkdir -p /var/lib/kubernetes/

  sudo mv ca.pem ca-key.pem master-kubernetes-key.pem master-kubernetes.pem \
    service-account-key.pem service-account.pem \
    encryption-config.yaml /var/lib/kubernetes/
}
```
![AWS Solution](./self_study/images/m6.png)

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

```bash
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

export INTERNAL_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/local-ipv4)
```

__Create the kube-apiserver.service systemd unit file:__

[Ensure to read each startup flag used in below systemd file from the documentation here](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/master-kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --etcd-servers=https://172.31.0.10:2379,https://172.31.0.11:2379,https://172.31.0.12:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/master-kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --runtime-config="api/all=true" \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${INTERNAL_IP}:6443 \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/master-kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

5. Configure the Kubernetes Controller Manager:

Move the kube-controller-manager kubeconfig into place:

```bash
sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Export some variables to retrieve the `vpc_cidr` - This will be required for the `bind-address` flag:

```bash
# Export the metadata base URL
export AWS_METADATA="http://169.254.169.254/latest/meta-data"

# Generate IMDSv2 token
TOKEN=$(curl -X PUT "$AWS_METADATA/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Retrieve the first MAC address
export EC2_MAC_ADDRESS=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s "$AWS_METADATA/network/interfaces/macs/" | head -n1 | tr -d '/')

# Retrieve the VPC CIDR block using the MAC address
export VPC_CIDR=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s "$AWS_METADATA/network/interfaces/macs/$EC2_MAC_ADDRESS/vpc-ipv4-cidr-block")

export NAME=k8s-cluster-from-ground-up
```

__Create the kube-controller-manager.service systemd unit file:__

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=${VPC_CIDR} \\
  --cluster-name=${NAME} \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

6. Configure the Kubernetes Scheduler

Move the kube-scheduler kubeconfig into place:

```bash
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
sudo mkdir -p /etc/kubernetes/config
```

__Create the kube-scheduler.yaml configuration file__

```bash
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```
![AWS Solution](./self_study/images/m7.png)

__Create the kube-scheduler.service systemd unit file__

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
![AWS Solution](./self_study/images/m9.png)

7. Start the Controller Services

```bash
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```
__Check the status of the services.__

Start with the `kube-scheduler` and `kube-controller-manager`. It may take up to 20 seconds for `kube-apiserver` to be fully loaded.

```bash
{
  sudo systemctl status kube-scheduler
  sudo systemctl status kube-controller-manager
  sudo systemctl status kube-apiserver
}
```
![AWS Solution](./self_study/images/m12.png)

__Observation:__ The `api-server` failed to start up on the server. As DevOps engineers, we must be able to solve problems.

__HINTS:__

i. The problem relates to `etcd` configuration.
ii. Check the systemd logs for the `api-server`. The problem will be clearly logged, and it will give you an idea what is wrong. Find out how to fix it.

__Troubleshooting__

__Check the systemd logs for the `api-server`__

Use `journalctl` to view detailed logs for the kube-apiserver service:
```bash
sudo journalctl -u kube-apiserver.service -b
```
Here's the error found in the log
```bash
Error: error while parsing encryption provider configuration file "/var...
```
This error suggests there is a misconfiguration in the encryption provider file, located at `/var/...`. (/var/lib/kubernetes/encryption-config.yaml).

Check the EncryptionConfig YAML file (encryption-config.yaml) in the Kubernetes setup, which configures how resources (such as secrets) are encrypted in `etcd`. The configuration specified the use of `AES` (Advanced Encryption Standard) specifically `aescbc`.

We have generated a 64-byte key, which is quite large. Typically, an AES encryption key is either 16, 24, or 32 bytes (128, 192, or 256 bits).
Therefore, we need to ensure the correct length by adjusting the size.

- __For AES-256 encryption__, the key should be 32 bytes:

```bash
ETCD_ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
This expression generates a base64-encoded encryption key with a size of 32 bytes.
After fixing the error, check the api-server status again:

```bash
sudo systemctl status kube-apiserver
```
![AWS Solution](./self_study/images/m14.png)


# Test that Everything is working fine

1. To get the cluster details run:

```bash
kubectl cluster-info  --kubeconfig admin.kubeconfig
```
OUTPUT:
```
Kubernetes control plane is running at https://k8s-api-server.svc.darey.io:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
![AWS Solution](./self_study/images/yem.png)

2. To get the current namespaces:

```bash
kubectl get namespaces --kubeconfig admin.kubeconfig
```
OUTPUT:
```
NAME              STATUS   AGE
default           Active   22m
kube-node-lease   Active   22m
kube-public       Active   22m
kube-system       Active   22m
```
![AWS Solution](./self_study/images/yem1.png)

3. To reach the Kubernetes API Server publicly

```bash
curl --cacert /var/lib/kubernetes/ca.pem https://$INTERNAL_IP:6443/version
```
OUTPUT:

```bash
{
  "major": "1",
  "minor": "21",
  "gitVersion": "v1.21.0",
  "gitCommit": "cb303e613a121a29364f75cc67d3d580833a7479",
  "gitTreeState": "clean",
  "buildDate": "2021-04-08T16:25:06Z",
  "goVersion": "go1.16.1",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```
![AWS Solution](./self_study/images/yem3.png)

4. To get the status of each component:

```bash
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```
![AWS Solution](./self_study/images/yem4.png)

![AWS Solution](./self_study/images/yem5.png)

5. On one of the controller nodes, configure __Role Based Access Control__ (RBAC) so that the api-server has necessary authorization for for the kubelet.

- Create the __ClusterRole__:

```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```
![AWS Solution](./self_study/images/yem5.png)

- Create the __ClusterRoleBinding__ to bind the kubernetes user with the role created above

```bash
cat <<EOF | kubectl --kubeconfig admin.kubeconfig  apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```
![AWS Solution](./self_study/images/yem6.png)


# Configuring the Kubernetes Worker nodes

Before we begin to bootstrap the worker nodes, it is important to understand that the K8s API Server authenticates to the __kubelet__ as the __kubernetes__ user using the same `kubernetes.pem` certificate.

We need to configure __Role Based Access__ (RBAC) for Kubelet Authorization

1. Configure `RBAC` permissions to allow the `Kubernetes API Server` to access the `Kubelet API` on each worker node. Access to the Kubelet API is required for retrieving metrics, logs, and executing commands in pods.

Create the `system:kube-apiserver-to-kubelet` __ClusterRole__ with permissions to access the Kubelet API and perform most common tasks associated with managing pods on the worker nodes.

Run the below script on the Controller node:

```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```
![AWS Solution](./self_study/images/yem7.png)

2. Bind the `system:kube-apiserver-to-kubelet` __ClusterRole__ to the `kubernetes` user so that API server can authenticate successfully to the `kubelets` on the worker nodes:

```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```
![AWS Solution](./self_study/images/yem8.png)

# Bootstraping components on the worker nodes

The following components will be installed on each node:

- kubelet
- kube-proxy
- Containerd or Docker
- Networking plugins

### 1. SSH into the worker nodes

__Worker-1__

```bash
worker_1_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-worker-0" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_1_ip}
```

__Worker-2__

```bash
worker_2_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-worker-1" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_2_ip}
```

__Worker-3__

```bash
worker_3_ip=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=${NAME}-worker-2" \
  --output text --query 'Reservations[].Instances[].PublicIpAddress')
  ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_3_ip}
```

### 2. Install OS dependencies:

```bash
{
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
}
```
![AWS Solution](./self_study/images/wo1.png)

### More about the dependencies:

- __`socat`__. Socat is the default implementation for Kubernetes port-forwarding when using __dockershim__ for the kubelet runtime. You will get to experience port-forwarding with Kubernetes in the next project. But what is Dockershim?

- __`Dockershim`__ was a temporary solution proposed by the Kubernetes community to add support for Docker so that it could serve as its container runtime. You should always remember that Kubernetes can use different container runtime to run containers inside its `pods`. For many years, Docker has been adopted widely and has been used as the container runtime for kubernetes. Hence the implementation that allowed docker is called the `Dockershim`. If you [check the source code of Dockershim](https://github.com/kubernetes/kubernetes/blob/770d3f181c5d7ed100d1ba43760a74093fc9d9ef/pkg/kubelet/dockershim/docker_streaming_others.go#L42), you will see that `socat` was used to implement the port-forwarding functionality.

- __`conntrack`__ Connection tracking (“conntrack”) is a core feature of the Linux kernel's networking stack. It allows the kernel to keep track of all logical network connections or flows, and thereby identify all of the packets which make up each flow so they can be handled consistently together. It is essential for performant complex networking of Kubernetes where nodes need to track connection information between thousands of pods and services.

- __`ipset`__ is an extension to iptables which is used to configure firewall rules on a Linux server. ipset is a module extension to iptables that allows firewall configuration on a "set" of IP addresses. Compared with how iptables does the configuration linearly, ipset is able to store sets of addresses and index the data structure, making lookups very efficient, even when dealing with large sets. Kubernetes uses ipsets to implement a distributed firewall solution that enforces network policies within the cluster. This can then help to further restrict communications across pods or namespaces. For example, if a namespace is configured with DefaultDeny isolation type (Meaning no connection is allowed to the namespace from another namespace), network policies can be configured in the namespace to whitelist the traffic to the pods in that namespace.

__Quick Overview Of Kubernetes Network Policy And How It Is Implemented__

Kubernetes network policies are application centric compared to infrastructure/network centric standard firewalls. There are no explicit CIDR or IP used for matching source or destination IP’s. Network policies build up on labels and selectors which are key concepts of Kubernetes that are used for proper organization (for e.g dedicating a namespace to data layer and controlling which app is able to connect there). A typical network policy that controls who can connect to the database namespace will look like below:

```bash
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  name: database-network-policy
  namespace: tooling-db
spec:
  podSelector:
    matchLabels:
      app: mysql
  ingress:
   - from:
     - namespaceSelector:
       matchLabels:
         app: tooling
     - podSelector:
       matchLabels:
       role: frontend
   ports:
     - protocol: tcp
     port: 3306
```

__NOTE:__ Best practice is to use solutions like `RDS` for database implementation. So the above is just to help you understand the concept.


### 3. Disable Swap

If [swap](https://opensource.com/article/18/9/swap-space-linux-systems) is not disabled, kubelet will not start. It is highly recommended to allow Kubernetes to handle resource allocation.

Test if swap is already enabled on the host:

```bash
sudo swapon --show
```

If there is no output, then you are good to go. Otherwise, run below command to turn it off

```bash
sudo swapoff -a
```

### 4. Download and install a container runtime. (Docker Or Containerd)

Before you install any container runtime, you need to understand that Docker is now deprecated, and Kubernetes no longer supports the Dockershim codebase from `v1.20 release`

Read more about this notice [here](https://kubernetes.io/blog/2020/12/02/dockershim-faq/)

If you install Docker, it will work. But be aware of this huge change.

- __Docker__

```bash
sudo apt update -y && \
sudo apt -y install apt-transport-https ca-certificates curl software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" && \
sudo apt update -y && \
apt-cache policy docker-ce && \
sudo apt -y install docker-ce && \
sudo usermod -aG docker ${USER} && \
sudo systemctl status docker
```

__NOTE:__ _exit the shell and log back in. Otherwise, you will face a permission denied error. Alternatively, you can run `newgrp docker` without exiting the shell. But you will need to provide the password of the logged in user_

- __Containerd__

Download binaries for `runc`, `cri-ctl`, and `containerd`

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.0.0-rc93/runc.amd64 \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.21.0/crictl-v1.21.0-linux-amd64.tar.gz \
  https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-linux-amd64.tar.gz
```
![AWS Solution](./self_study/images/wo2.png)

__Configure containerd:__

```bash
{
  mkdir containerd
  tar -xvf crictl-v1.21.0-linux-amd64.tar.gz
  tar -xvf containerd-1.4.4-linux-amd64.tar.gz -C containerd
  sudo mv runc.amd64 runc
  chmod +x  crictl runc
  sudo mv crictl runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
}
```
![AWS Solution](./self_study/images/wo3.png)

```bash
sudo mkdir -p /etc/containerd/
```

```bash
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF
```
![AWS Solution](./self_study/images/wo4.png)

Create the __containerd.service__ systemd unit file:

```bash
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```
![AWS Solution](./self_study/images/wo5.png)

### 5. Create directories to configure `kubelet`, `kube-proxy`, `cni`, and a directory to keep the `kubernetes` `root` `ca` file:

```bash
sudo mkdir -p \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubernetes \
  /var/run/kubernetes
```
### 6. Download and Install CNI

CNI (Container Network Interface), a [Cloud Native Computing Foundation project](https://www.cncf.io/), consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers. It also comes with a number of plugins.

Kubernetes uses CNI as an interface between network providers and Kubernetes Pod networking. Network providers create network plugin that can be used to implement the Kubernetes networking, and includes additional set of rich features that Kubernetes does not provide out of the box.

Download the plugins available from [containernetworking's](https://github.com/containernetworking/cni) GitHub repo and read more about CNIs and why it is being developed.

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz
```
Install CNI into `/opt/cni/bin/`

```bash
sudo tar -xvf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin/
```
The output shows the plugins that comes with the CNI.

```
./
./macvlan
./flannel
./static
./vlan
./portmap
./host-local
./vrf
./bridge
./tuning
./firewall
./host-device
./sbr
./loopback
./dhcp
./ptp
./ipvlan
./bandwidth
```
![AWS Solution](./self_study/images/wo6.png)

There are few other plugins that are not included in the CNI, which are also widely used in the industry. They all have their unique implementation approach and set of features.

Click to read more about each of the network plugins below:

- [Calico](https://www.tigera.io/project-calico/)
- [Weave Net](https://www.weave.works/)
- [flannel](https://github.com/flannel-io/flannel)
[source](https://ranchermanager.docs.rancher.com/v2.5/faq/container-network-interface-providers)


Sometimes you can combine more than one plugin together to maximize the use of features from different providers. Or simply use a CNI network provider such as [canal](https://github.com/projectcalico/canal) that gives you the best of Flannel and Calico.

### 7. Download binaries for kubectl, kube-proxy, and kubelet

```bash
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet
```
![AWS Solution](./self_study/images/wo7.png)

### 8. Install the downloaded binaries

```bash
{
  chmod +x  kubectl kube-proxy kubelet
  sudo mv  kubectl kube-proxy kubelet /usr/local/bin/
}
```
__Configure the worker nodes components__

### 9. Configure `kubelet`:

In the home directory, you should have the certificates and `kubeconfig` file for each node. A list in the home folder should look like below:

![AWS Solution](./self_study/images/wo8.png)

__Configuring the network__

Get the `POD_CIDR` that will be used as part of network configuration

```bash
POD_CIDR=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^pod-cidr" | cut -d"=" -f2)

echo "${POD_CIDR}"
```
![AWS Solution](./self_study/images/wo9.png)

In case you are wondering where this `$POD_CIDR` is coming from. Well, this was configured at the time of creating the worker nodes. Remember the for loop below? The `--user-data` flag is where we specified what we want the POD_CIDR to be. It is very important to ensure that the CIDR does not overlap with EC2 IPs within the subnet. In the real world, this will be decided in collaboration with the Network team.

Why do we need a network plugin? And why network configuration is crucial to implementing a Kubernetes cluster?

First, let us understand the Kubernetes networking model:

The networking model assumes a flat network, in which containers and nodes can communicate with each other. That means, regardless of which node is running the container in the cluster, Kubernetes expects that all the containers must be able to communicate with each other. Therefore, any network interface used for a Kubernetes implementation must follow this requirement. Otherwise, containers running in pods will not be able to communicate. Of course, this has security concerns. Because if an attacker is able to get into the cluster through a compromised container, then the entire cluster can be exploited.

To mitigate security risks and have a better controlled network topology, Kubernetes uses CNI (Container Network Interface) to manage Network Policies which can be used to operate the Pod network through external plugins such as Calico, Flannel or Weave Net to name a few. With these, you can set policies similar to how you would configure segurity groups in AWS and limit network communications through either cidr ipBlock, namespaceSelectors, or podSelectors, you will see more of these concepts further on.

To really understand Kubernetes further, let us explore some basic concepts around its networking:

__`Pods:`__

A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) is the basic building block of Kubernetes; it is the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents a running process on your cluster. It encapsulates a container running an application such as the __Tooling website__ (or, in some cases, multiple containers), storage resources, a unique network IP, and options that govern how the container(s) should run. All the containers running inside a Pod can reach each other on localhost.

For example, if you deploy both Tooling and MySQL containers inside the same pod, then both of them are considered running on localhost. Although this design pattern is not ideal. Most likely they will run in separate Pods. In most cases one Pod contains just one container, but there are some design patterns that imply multi-container pods (e.g. sidecar, ambassador, adapter) - you can read more about them in [this article](https://betterprogramming.pub/understanding-kubernetes-multi-container-pod-patterns-577f74690aee).

For a better understanding, of Kubernetes networking, let us assume that we have 2-containers in a single Pod and we have 2 such Pods (we can actually have as many pods of the same composition as our node resources would allow).

Network configuration will look like this:

![AWS Solution](./self_study/images/wo11.png)

Notice, that both containers share a single virtual network interface veth0 that belongs to a virtual network within a single node. This virtual interface veth0 is used to allow communication from a pod to the outer world through a bridge cbr0 (custom bridge). This bridge is an interface that forwards the traffic from the Pods on one node to other nodes through a physical network interface eth0. Routing between the nodes is done by means of a router with the routing table.

For more detailed explanation of different aspects of Kubernetes networking - watch [this video](https://www.youtube.com/watch?v=5cNrTU6o3Fw).

__Pod Network__

You must decide on the __Pod CIDR__ per worker node. Each worker node will run multiple pods, and each pod will have its own IP address. IP address of a particular Pod on worker node 1 should be able to communicate with the IP address of another particular Pod on worker node 2. For this to become possible, there must be a bridge network with virtual network interfaces that connects them all together. Here is an [interesting read that goes a little deeper into how it works](https://www.digitalocean.com/community/tutorials/kubernetes-networking-under-the-hood) _Bookmark that page and read it over and over again after you have completed this project_

### 10. Configure the bridge and loopback networks

__Bridge:__

```bash
cat > 172-20-bridge.conf <<EOF
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```
__Loopback:__

```bash
cat > 99-loopback.conf <<EOF
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```
![AWS Solution](./self_study/images/wo12.png)

### 11. Move the files to the network configuration directory:

```bash
sudo mv 172-20-bridge.conf 99-loopback.conf /etc/cni/net.d/
```

### 12. Store the worker's name in a variable:

```bash
NAME=k8s-cluster-from-ground-up

WORKER_NAME=${NAME}-$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo "${WORKER_NAME}"
```

### 13. Move the certificates and kubeconfig file to their respective configuration directories:

```bash
sudo mv ${WORKER_NAME}-key.pem ${WORKER_NAME}.pem /var/lib/kubelet/
sudo mv ${WORKER_NAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```
### 14. Create the `kubelet-config.yaml` file

Ensure the needed variables exist:

```bash
NAME=k8s-cluster-from-ground-up

WORKER_NAME=${NAME}-$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo "${WORKER_NAME}"
```
```bash
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
resolvConf: "/etc/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${WORKER_NAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${WORKER_NAME}-key.pem"
EOF
```
![AWS Solution](./self_study/images/wo12.png)

Let us talk about the configuration file kubelet-config.yaml and the actual configuration for a bit. Before creating the systemd file for __kubelet__, it is recommended to create the `kubelet-config.yaml` and set the configuration there rather than using multiple startup flags in systemd. You will simply point to the `yaml` file.

The config file specifies where to find `certificates`, the `DNS server`, and authentication information. As you already know, `kubelet` is responsible for the containers running on the node, regardless if the runtime is Docker or Containerd; as long as the containers are being created through Kubernetes, `kubelet` manages them. If you run any `docker` or `cri` commands directly on a worker to create a container, bear in mind that Kubernetes is not aware of it, therefore `kubelet` will not manage those. Kubelet's major responsibility is to always watch the containers in its care, by default every 20 seconds, and ensuring that they are always running. Think of it as a process watcher.

The `clusterDNS` is the address of the DNS server. As of Kubernetes v1.12, `CoreDNS` is the recommended DNS Server, hence we will go with that, rather than using legacy `kube-dns`.

__Note:__ _The `CoreDNS` Service is named `kube-dns`(When you see kube-dns, just know that it is using `CoreDNS`). This is more of a backward compatibility reasons for workloads that relied on the legacy `kube-dns` Service name._

In Kubernetes, `Pods` are able to find each other using service names through the internal DNS server. Every time a service is created, it gets registered in the DNS server.

In Linux, the `/etc/resolv.conf` file is where the DNS server is configured. If you want to use Google's public DNS server (8.8.8.8) your /etc/resolv.conf file will have following entry:

```
nameserver 8.8.8.8
```

In Kubernetes, the kubelet process on a worker node configures each pod. Part of the configuration process is to create the file `/etc/resolv.conf` and specify the correct DNS server.

### 15. Configure the `kubelet` systemd service

```bash
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service
[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --cluster-domain=cluster.local \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF
```

### 16. Create the `kube-proxy.yaml` file

```bash
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "172.31.0.0/16"
EOF
```
![AWS Solution](./self_study/images/wo14.png)

### 17. Configure the Kube Proxy systemd service

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF
```
![AWS Solution](./self_study/images/wo15.png)

### 18. Reload configurations and start both services

```bash
{
  sudo systemctl daemon-reload
  sudo systemctl enable containerd kubelet kube-proxy
  sudo systemctl start containerd kubelet kube-proxy
}
```
```bash
sudo systemctl status containerd
sudo systemctl status kubelet
sudo systemctl status kube-proxy
```
![AWS Solution](./self_study/images/ge3.png)

![AWS Solution](./self_study/images/ge2.png)

![AWS Solution](./self_study/images/ge1.png)

Now you should have the worker nodes joined to the cluster, and in a __READY__ state.

```bash
kubectl get nodes --kubeconfig admin.kubeconfig -o wide
```
![AWS Solution](./self_study/images/done.png)

__Troubleshooting Tips:__ If you have issues at this point. Consider the below:

1. Use journalctl -u <service name> to get the log output and read what might be wrong with starting up the service. You can redirect the output into a file and analyse it.
2. Review your PKI setup again. Ensure that the certificates you generated have the hostnames properly configured.
3. It is okay to start all over again. Each time you attempt the solution is an opportunity to learn something.

## Conclusion

In this project, we successfully orchestrated containers across multiple virtual servers using Kubernetes, building a production-like cluster from the ground up on AWS. We started by understanding the limitations of Docker Compose for scalable, high-availability environments and moved to Kubernetes, the industry-leading container orchestration platform. Throughout the process, we provisioned EC2 infrastructure, configured secure networking, created and distributed TLS certificates, implemented Role-Based Access Control (RBAC), and deployed critical components like the API server, etcd, kubelet, and kube-proxy. This hands-on work strengthened foundational knowledge of Kubernetes architecture and operations, while also reinforcing essential DevOps practices such as automation, infrastructure-as-code readiness, security, and troubleshooting. With this setup completed, you now have a strong base to build on, preparing you for more advanced Kubernetes topics like Helm, CI/CD pipelines, cluster monitoring, and service mesh integrations—and ultimately enabling you to manage containerized applications at scale with confidence.

