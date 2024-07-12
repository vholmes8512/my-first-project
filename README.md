# My First Terraform Project: Terraform Configuration for AWS VPC with Public and Private Subnets

I wanted to demonstrate my learning on Terraform by creating a Terraform configuration that sets up an AWS Virtual Private Cloud (VPC) with three public subnets, three private subnets, an Internet Gateway, and a NAT Gateway. The configuration also includes route tables and associations to ensure proper routing for both public and private subnets. I will use steps below to complete and test my terraform congfiguration:

1. Provider
2. VPC
3. Public Subnets
4. Private Subnets
5. Internet Gateway
6. NAT Gateway
7. Route Tables
8. Route Table Association

Below is a diagram showing the end state of the AWS environment:

![40660776-2C8E-4039-8472-66A4390E2145](https://github.com/user-attachments/assets/01ca1dae-e212-4a23-9336-1f83785ab394)

* Open Visual Studio and then create a folder named main.tf. This folder will contain the main set of configuration of your file for the deployment of the required resources.
* You can also find these templates for these resources on the Terraform Registry
under "hashicorp/aws"

Creating a VPC with three private subnets and three public subnets is a common architecture choice in AWS for several reasons:

1. High Availability and Fault Tolerance
Multi-AZ Deployment: By distributing the subnets across multiple Availability Zones, you ensure high availability. If one AZ goes down, the other AZs can still operate, maintaining the service's uptime.
Redundancy: Having multiple subnets in different AZs adds redundancy, reducing the risk of a single point of failure.
2. Separation of Public and Private Resources
Public Subnets: These are used for resources that need to be accessible from the internet, such as web servers, load balancers, and bastion hosts.
Private Subnets: These are used for resources that should not be directly accessible from the internet for security reasons, such as databases, application servers, and internal services.
3. Security
Isolation: By segregating public and private resources, you can apply more granular security controls, such as security groups and network ACLs, to control traffic flow.
NAT Gateway: Private subnets can access the internet for updates and patches via a NAT Gateway in a public subnet, without exposing the resources to inbound internet traffic.
4. Scalability
Resource Distribution: Distributing resources across multiple subnets and AZs allows for better load balancing and resource utilization.
Scalable Design: This design supports future scaling needs. As your application grows, you can add more subnets and resources without major architectural changes.
5. Compliance and Best Practices
AWS Best Practices: Following AWS best practices often involves separating public and private resources and deploying them across multiple AZs for redundancy and availability.
Compliance Requirements: Some industries have regulatory requirements for data isolation and security, which can be better achieved with this architecture.

# Provider
First, I will define the AWS provider and specify the region where our resources will be created.

```hcl
# Configure the AWS Provider
provider "aws" {
  region = "us-west-2"
}
```

# VPC
Next, I will create a Virtual Private Cloud (VPC) which is the main network container for all our subnets and gateways.

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "main-vpc"
  }
}
```
# Public Subnets
I will create three public subnets within the VPC. Public subnets have direct access to the Internet.

```hcl
resource "aws_subnet" "public_subnet_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-west-2a"

  tags = {
    Name = "public-subnet-1"
  }
}

resource "aws_subnet" "public_subnet_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-west-2b"

  tags = {
    Name = "public-subnet-2"
  }
}

resource "aws_subnet" "public_subnet_3" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "us-west-2c"

  tags = {
    Name = "public-subnet-3"
  }
}
```

# Private Subnets
I will create three private subnets within the VPC. Private subnets do not have direct access to the Internet but can communicate through the NAT Gateway.

```hcl
resource "aws_subnet" "private_subnet_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "us-west-2a"

  tags = {
    Name = "private-subnet-1"
  }
}

resource "aws_subnet" "private_subnet_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.5.0/24"
  availability_zone = "us-west-2b"

  tags = {
    Name = "private-subnet-2"
  }
}

resource "aws_subnet" "private_subnet_3" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.6.0/24"
  availability_zone = "us-west-2c"

  tags = {
    Name = "private-subnet-3"
  }
}
```

# Internet Gateway
I will create an Internet Gateway to allow the public subnets to communicate with the Internet.

```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}
```
# NAT Gateway
I will create a NAT Gateway to enable instances in the private subnets to connect to the Internet for updates and patches, while still being protected from inbound traffic.

```hcl
resource "aws_eip" "nat" {
  domain = "vpc"
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public_subnet_1.id

  tags = {
    Name = "main-nat-gateway"
  }
}
```
# Route Tables
I will create route tables for both public and private subnets to define how traffic is routed.

Public Route Table: Routes all traffic (0.0.0.0/0) to the Internet Gateway.
Private Route Table: Routes all traffic (0.0.0.0/0) to the NAT Gateway.

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "public-route-table"
  }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = {
    Name = "private-route-table"
  }
}
```

# Route Table Associations
Finally, I will associate the public and private subnets with their respective route tables.

```hcl
resource "aws_route_table_association" "public_subnet_1" {
  subnet_id      = aws_subnet.public_subnet_1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_subnet_2" {
  subnet_id      = aws_subnet.public_subnet_2.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_subnet_3" {
  subnet_id      = aws_subnet.public_subnet_3.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private_subnet_1" {
  subnet_id      = aws_subnet.private_subnet_1.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private_subnet_2" {
  subnet_id      = aws_subnet.private_subnet_2.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private_subnet_3" {
  subnet_id      = aws_subnet.private_subnet_3.id
  route_table_id = aws_route_table.private.id
}
```

# Usage
1. Initialize Terraform:

```sh
terraform init
```

2. Apply the configuration

```sh
terraform apply
```
In conclusion, this will set up your AWS environment with the specified VPC, subnets and routing. Before running commands, make sure you have your AWS credentials configured properly. Feel feel to try out this project. It's a great way to understand the objective for each resource used in the written code.


