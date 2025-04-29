# AWS VPC Infrastructure Setup

This guide walks through the step-by-step process to set up a secure and scalable VPC infrastructure on AWS, including subnets, routing, security groups, EC2 instances, and load balancing.

---

## Step 1: Create a VPC

1. Go to the **VPC Console** in AWS.
2. Click **Create VPC**.
3. Configure the following:
   - **Name**: `MyVPC`
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - **Tenancy**: `Default`
4. Click **Create VPC**.

---

## Step 2: Create Subnets

### Private Subnets
- **PrivateSubnet1**
  - CIDR block: `10.0.1.0/24`
  - Availability Zone: `us-east-1a`
- **PrivateSubnet2**
  - CIDR block: `10.0.2.0/24`
  - Availability Zone: `us-east-1b`

### Public Subnets
- **PublicSubnet1**
  - CIDR block: `10.0.3.0/24`
  - Availability Zone: `us-east-1a`
- **PublicSubnet2**
  - CIDR block: `10.0.4.0/24`
  - Availability Zone: `us-east-1b`

---

## Step 3: Create an Internet Gateway (IGW)

1. Go to **Internet Gateways**.
2. Click **Create Internet Gateway**.
   - **Name**: `MyIGW`
3. Attach the IGW to `MyVPC`.

---

## Step 4: Create a NAT Gateway

1. Go to **NAT Gateways**.
2. Click **Create NAT Gateway**.
   - **Subnet**: `PublicSubnet1`
   - **Elastic IP**: Allocate a new one
3. Click **Create NAT Gateway**.

---

## Step 5: Configure Route Tables

### Public Route Table
- Create a new route table.
- Add route:
  - **Destination**: `0.0.0.0/0`
  - **Target**: Internet Gateway (MyIGW)
- Associate with `PublicSubnet1` and `PublicSubnet2`.

### Private Route Table
- Create a new route table.
- Add route:
  - **Destination**: `0.0.0.0/0`
  - **Target**: NAT Gateway
- Associate with `PrivateSubnet1` and `PrivateSubnet2`.

---

## Step 6: Create Security Groups

### Application Server SG
- Inbound: Allow `HTTP (80)`, `HTTPS (443)`
- Inbound: Allow traffic from ALB SG
- Outbound: Allow traffic to DB server on port `3306`

### Database Server SG
- Inbound: Allow traffic on port `3306` from App SG
- Outbound: Allow access to S3

### VPN Server SG
- Inbound: Allow traffic on port `1194` (OpenVPN)
- Outbound: Allow to private subnets

### ALB Security Group
- Inbound: Allow `HTTP (80)`, `HTTPS (443)`
- Outbound: Allow to Application Server SG

---

## Step 7: Create Application Load Balancer (ALB)

1. Go to **Load Balancers**.
2. Click **Create Load Balancer** → Application Load Balancer.
3. Configure:
   - **Name**: `MyALB`
   - **Scheme**: Internet-facing
   - **Listeners**: `HTTP (80)`, `HTTPS (443)`
   - **Availability Zones**: `PublicSubnet1`, `PublicSubnet2`
4. Create a **Target Group** for app servers and register them.

---

## Step 8: Launch EC2 Instances

### Application Server
- AMI: Amazon Linux 2
- Type: `t2.micro` or larger
- Subnet: `PrivateSubnet1`
- SG: Application Server SG

### Database Server
- AMI: Amazon Linux 2 or RDS
- Type: `t2.micro` or larger
- Subnet: `PrivateSubnet2`
- SG: Database Server SG

### VPN Server
- AMI: OpenVPN Access Server
- Type: `t2.micro` or larger
- Subnet: `PublicSubnet1`
- SG: VPN Server SG

---

## Step 9: Configure Auto Scaling

1. Create an **Auto Scaling Group** for app servers.
2. Use the same launch config as the app server.
3. Subnets: `PrivateSubnet1`, `PrivateSubnet2`
4. Target Group: Attach the ALB target group.
5. Define scaling policies for min/max/desired instances.

---

## Step 10: Create an S3 Bucket

1. Go to **S3 Console**.
2. Click **Create Bucket**.
   - **Name**: `my-app-database-bucket`
   - **Region**: Same as VPC
3. Set permissions to allow access from app and DB SGs.

---

## Step 11: Configure Network ACLs (NACLs)

### Public NACL
- Allow inbound/outbound `HTTP`, `HTTPS`, `VPN` traffic

### Private NACL
- Allow traffic for `Database` (port 3306) and `S3`

---

## Step 12: Test the Setup

- Deploy your application on the **application server**
- Ensure connection to the **database server**
- Test **S3 access** from app and DB servers
- Verify **ALB routing and Auto Scaling**

---

## Notes

- Be sure to use appropriate IAM roles and policies where necessary.
- Enable CloudWatch for monitoring instance health and scaling.
- Secure all resources following AWS security best practices.
