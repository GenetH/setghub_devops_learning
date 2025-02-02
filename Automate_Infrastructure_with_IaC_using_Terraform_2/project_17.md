# Automate Infrastructure With IaC using Terraform 2
---

Before automating infrastructure with Terraform, having a foundational understanding of networking concepts is crucial. This includes learning about **IP Addresses**, **Subnets**, **CIDR Notation**, and **Networking Protocols**.

#### **Key Resources**
1. **Eli the Computer Guy Videos**:
   - [Introduction to Networking](https://www.youtube.com/watch?v=rL8RSFQG8do)
   - [TCP/IP and Subnet Masking](https://www.youtube.com/watch?v=EkNq4TrHP_U)
2. **Digital Ocean Articles** by Justin Ellingwood:
   - Networking Terminology
   - Subnets and CIDR Notation

**Justin Ellingwood blog posts**

WARNING: You may initially feel overwhelmed by the information provided in these articles. It’s alright if We do not fully understand them the first time. Bookmark the pages, read them again and again for the next few days, find other articles talking about the same topics on Google and watch YouTube videos about them. Subconsciously, you will begin to understand them over time.

- [Networking Part 1](https://www.digitalocean.com/community/tutorials/an-introduction-to-networking-terminology-interfaces-and-protocols)
- [Networking Part 2](https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking#netmasks-and-subnets)

**Continue Infrastructure Automation with Terraform**  
Let us continue from where we stopped in the previous project.  

Based on my knowledge from the previous project, I can continue creating AWS resources!  

### **Networking: Private Subnets & Best Practices**

When creating **4 private subnets**, follow these best practices:

1. **Use Variables or the `length()` Function**:
   - Determine the number of Availability Zones (AZs) dynamically.
   
2. **Use the `cidrsubnet()` Function**:
   - Allocate VPC CIDR blocks dynamically for subnets.

3. **Keep Variables and Resources Organized**:
   - Store variables in separate files for better code structure and readability.

4. **Apply Resource Tagging**:
   - Use tags to manage and organize our resources efficiently. Explore functions like `format()` and `count` for dynamic tagging.

![AWS Solution](./self_study/images/a.png)
![AWS Solution](./self_study/images/b.png)

### **Tagging: Importance and Implementation**

Tagging is a straightforward yet powerful way to organize your resources. It helps with:

- Better resource organization into "virtual groups."
- Easier filtering and searching, both programmatically and via the console.
- Simplified billing and cost management (e.g., by department or environment).
- Identifying unused resources for cleanup.
- Collaboration across teams using a shared AWS account.

### **Example: Setting Tags in Terraform**

#### **Static Tag Example:**
```hcl
tags = {
  Environment     = "production"
  Owner-Email     = "admin@example.com"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}
```
#### **Dynamic Tag Example Using `merge()`:**
```hcl
tags = merge(
  var.default_tags,
  {
    Name = "Private Subnet ${count.index}"
  }
)
```

**Explanation:**
- Use the `merge()` function to combine default tags with resource-specific tags dynamically.
- This approach allows changes to be made in a single location for consistent updates across all resources.

### Internet Gateway Creation with `format()` Function

Create an **Internet Gateway** in a separate Terraform file `internet_gateway.tf`
```hcl
resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s!", aws_vpc.main.id, "IG")
    }
  )
}
```
![AWS Solution](./self_study/images/c.png)
![AWS Solution](./self_study/images/d.png)

Did you notice how we have used `format()` function to dynamically generate a unique name for this resource? The first part of the `%s` takes the interpolated value of `aws_vpc.main.id` while the second `%s` appends a literal string `IG` and finally an exclamation mark is added in the end.

If any of the resources being created is either using the `count` function or creating multiple resources using a loop, then a key-value pair that needs to be unique must be handled differently.

For example, each of our subnets should have a unique name in the tag section. Without the `format()` function, this would not be possible. With the `format` function, each private subnet’s tag will look like this:

```
Name = PrivateSubnet-0
Name = PrivateSubnet-1
Name = PrivateSubnet-2
```

Lets try and see that in action.

```hcl
tags = merge(
  var.tags,
  {
    Name = format("PrivateSubnet-%s", count.index)
  }
)
```

### NAT Gateways

Create 1 NAT Gateways and 1 Elastic IP (EIP) addresses.

Now use similar approach to create the NAT Gateways in a new file called natgateway.tf.

Note: We need to create an Elastic IP for the NAT Gateway, and you can see the use of depends_on to indicate that the Internet Gateway resource must be available before this should be created. Although Terraform does a good job to manage dependencies, in most cases, it is good to be explicit.

You can read more on dependencies [here](https://developer.hashicorp.com/terraform/language/meta-arguments/depends_on).
```hcl
resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}
```
![AWS Solution](./self_study/images/e.png)
![AWS Solution](./self_study/images/f.png)
![AWS Solution](./self_study/images/g.png)

### AWS Routes

To set up routes for both public and private subnets in Terraform, you can create a file called `route_tables.tf`. In this file, We'll define resources for the following:

- **aws_route_table**: For route tables.
- **aws_route**: For routing rules.
- **aws_route_table_association**: To associate subnets with route tables.
```hcl
# Create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table", var.name)
    },
  )
}

# Associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count        = length(aws_subnet.private[*].id)
  subnet_id    = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

# Create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table", var.name)
    },
  )
}

# Create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# Associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count        = length(aws_subnet.public[*].id)
  subnet_id    = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}
```
![AWS Solution](./self_study/images/h.png)
![AWS Solution](./self_study/images/i.png)
![AWS Solution](./self_study/images/j.png)
![AWS Solution](./self_study/images/k.png)


#### Output:

After running `terraform plan` and `terraform apply`, Terraform will provision the following in Our AWS environment:

- [x] Our Main VPC
- [x] 2 Public Subnets
- [x] 4 Private Subnets
- [x] 1 Internet Gateway
- [x] 1 NAT Gateway
- [x] 1 Elastic IP (EIP)
- [x] 2 Route Tables
![AWS Solution](./self_study/images/l.png)
This setup completes the networking part of AWS infrastructure. Now, we can move forward with Compute and Access Control configurations.

---

### AWS Identity and Access Management  
#### IAM and Roles

We want to pass an IAM role or EC2 instances to give them access to some specific resources, so we need to do the following:  

**1. Create AssumeRole**  
Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you may not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use **AssumeRole** within your account or for cross-account access.  

Add the following code to a new file named `roles.tf`:  

```hcl
resource "aws_iam_role" "ec2_instance_role" {
  name               = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}
```

In this code, we are creating **AssumeRole** with **AssumeRole policy**. It grants to an entity, in our case it is an EC2, permissions to assume the role.

**2. Create IAM Policy for this role**  
This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform an action `describe` applied to EC2 instances:

```hcl
resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy      = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume policy"
    },
  )
}
```
**3. Attach the Policy to the IAM Role**  
This is where we will be attaching the policy, which we created above, to the role we created in the first step.

```hcl
resource "aws_iam_role_policy_attachment" "test-attach" {
  role       = aws_iam_role.ec2_instance_role.name
  policy_arn = aws_iam_policy.policy.arn
}
```
**"4. Create an [Instance Profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) and associate it with the IAM Role."**
```hcl
resource "aws_iam_instance_profile" "ip" {
  name = "aws_instance_profile_test"
  role = aws_iam_role.ec2_instance_role.name
}
```
![AWS Solution](./self_study/images/ab.png)
![AWS Solution](./self_study/images/ac.png)

We are pretty much done with the Identity and Management part for now. Let us move on and create other resources required.  

---

### **Resources to be Created**
1. Create Security Groups.
2. Create Target Group for Nginx, WordPress, and Tooling.
3. Create Certificate from AWS Certificate Manager.
4. Create an External Application Load Balancer and Internal Application Load Balancer.
5. Create a Launch Template for Bastion, Tooling, Nginx, and WordPress.
6. Create an Auto Scaling Group (ASG) for Bastion, Tooling, Nginx, and WordPress.
7. Create an Elastic Filesystem.
8. Create Relational Database (RDS).

---

### **1. Create Security Groups**
We are going to create all the security groups in a single file. Then, we are going to reference these security groups within each resource that needs it.

- Check out the Terraform documentation for [Security Groups](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group).
- Check out the Terraform documentation for [Security Group Rules](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule).

Create a file and name it `security.tf`, and copy and paste the code below:

```hcl
# Security group for ALB to allow access from anywhere for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "ext-alb-sg"
    }
  )
}

# Security group for Bastion
resource "aws_security_group" "bastion_sg" {
  name        = "vpc_web_sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    }
  )
}

# Security group for Nginx Reverse Proxy
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    }
  )
}

# AWS Security Group Rules
resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

# Security group for Internal ALB
resource "aws_security_group" "int-alb-sg" {
  name   = "my-alb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-alb-sg"
    }
  )
}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}

# Security group for Webservers
resource "aws_security_group" "webserver-sg" {
  name   = "my-asg-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    }
  )
}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

# Security group for Datalayer
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    }
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}
```
![AWS Solution](./self_study/images/sa.png)
![AWS Solution](./self_study/images/sb.png)

Certainly! Here's the full content from the images you provided:

---

### 2. Create Certificate From Amazon Certificate Manager

Create a file named **cert.tf** and add the following code snippets to it.

**NOTE**: Read through to change the domain name to your own domain name including any other name that needs to be changed.

```hcl
# The entire section creates a certificate, public zone, and validates the certificate using DNS method.

# Create the certificate using a wildcard for all the domains created in your_domain.gq
resource "aws_acm_certificate" "your_domain" {
  domain_name       = "*.your_domain.gq"
  validation_method = "DNS"
}

# Calling the hosted zone
data "aws_route53_zone" "your_domain" {
  name         = "your_domain.gq"
  private_zone = false
}

# Selecting validation method
resource "aws_route53_record" "your_domain" {
  for_each = {
    for dvo in aws_acm_certificate.your_domain.domain_validation_options : dvo.domain_name => {
      name    = dvo.resource_record_name
      record  = dvo.resource_record_value
      type    = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.your_domain.zone_id
}

# Validate the certificate through DNS method
resource "aws_acm_certificate_validation" "your_domain" {
  certificate_arn        = aws_acm_certificate.your_domain.arn
  validation_record_fqdns = [for record in aws_route53_record.your_domain : record.fqdn]
}

# Create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.your_domain.zone_id
  name    = "tooling.your_domain.gq"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

# Create records for WordPress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.your_domain.zone_id
  name    = "wordpress.your_domain.gq"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```
![AWS Solution](./self_study/images/fa.png)
![AWS Solution](./self_study/images/fb.png)

### 3. Create an external (Internet-facing) Application Load Balancer (ALB)

Create a file named `alb.tf`

First of all, we will create the ALB, after which we will create the target group, and then, we will create the listener rule.

Useful Terraform DocumentationGo through this documentation and understand the arguments needed for each resource:
- [ALB](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb)
- [ALB Target Group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group)
- [ALB Listener](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_listener)

We need to create an ALB to balance the traffic between the instances:
```hcl
resource "aws_lb" "ext-alb" {
  name               = "ext-alb"
  internal           = false
  security_groups    = [aws_security_group.ext-alb-sg.id]
  subnets            = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "ACS-ext-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
```
![AWS Solution](./self_study/images/x.png)

To inform our ALB where to route the traffic, we need to create a [Target Group](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) to point to its targets:
```hcl
resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}
```
![AWS Solution](./self_study/images/xa.png)

Then we will need to create a [Listener](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listeners.html) for this target group.
```hcl
resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.oyindamola.certificate_arn

  default_action {
    type = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}
```
![AWS Solution](./self_study/images/xb.png)

Add the following outputs to `output.tf` to print them on screen:
```hcl
output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}
```
![AWS Solution](./self_study/images/xc.png)


### Create an Internal (Internal) Application Load Balancer (ALB)

For the Internal Load balancer, we will follow the same concepts as with the external load balancer.

Add the code snippets inside the **`alb.tf`** file:

```hcl
# -----------------------------
# Internal Load Balancers for webservers
# -----------------------------

resource "aws_lb" "ialb" {
  name               = "ialb"
  internal           = true
  security_groups    = [
    aws_security_group.int-alb-sg.id,
  ]
  subnets            = [
    aws_subnet.private[0].id,
    aws_subnet.private[1].id,
  ]

  tags = merge(
    var.tags,
    {
      Name = "ACS-int-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
``` 
![AWS Solution](./self_study/images/xd.png)