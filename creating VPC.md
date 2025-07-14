# 🛠️ AWS VPC Setup: Public & Private Subnets in 2 AZs

## 1. Create the VPC

- **Name:** `my-vpc`
- **CIDR Block:** `10.0.0.0/16`
- **Tenancy:** Default

Creates the base network with ~65,536 IPs.

---

## 2. Create 4 Subnets

| Name              | AZ           | CIDR Block    | Auto-assign Public IP |
|------------------|--------------|---------------|------------------------|
| public-subnet-1  | ap-south-1a  | 10.0.1.0/24   | ✅ Enabled             |
| public-subnet-2  | ap-south-1b  | 10.0.2.0/24   | ✅ Enabled             |
| private-subnet-1 | ap-south-1a  | 10.0.3.0/24   | ❌ Disabled            |
| private-subnet-2 | ap-south-1b  | 10.0.4.0/24   | ❌ Disabled            |

> Enable Auto-assign Public IP only for public subnets:
> - Subnets → Actions → Modify auto-assign IP settings
> - Enabling Auto-assign Public IP ensures that any EC2 instance launched in the public subnet automatically receives a public IPv4 address. Without it, the instance won't have internet access—even though it's in a public subnet with an IGW.

---

## 3. Create Internet Gateway (IGW)

- **Name:** `my-igw`
- **Attach to VPC:** `my-vpc`

Required for public subnet internet access.

---

## 4. Create Route Tables

### Public Route Table

- **Name:** `public-rt`
- **Routes:**
  - `0.0.0.0/0` → `igw-xxxxxxx`
- **Associate with Subnets:**
  - `public-subnet-1`, `public-subnet-2`

### Private Route Table

- **Name:** `private-rt`
- **Routes:**
  - Initially: `local` only
- **Associate with Subnets:**
  - `private-subnet-1`, `private-subnet-2`

---

## 5. Create NAT Gateway (with Elastic IP)

- Go to **VPC → NAT Gateways → Create NAT Gateway**
- **Subnet:** `public-subnet-1`
- ✅ Check **Allocate Elastic IP automatically**
- **Name:** `nat-gw`

Used for private subnet outbound internet access.

---

## 6. Update Private Route Table

- Go to **Route Tables → private-rt → Edit routes**
- Add Route:
  - `0.0.0.0/0` → `nat-gw-id`

Now private subnets can access the internet via NAT.

---

## 7. Launch EC2 Instances

### Public EC2 Instance

- Subnet: `public-subnet-1` or `public-subnet-2`
- Public IP: ✅ Auto-assigned (from subnet setting)
- Security Group:
  - Inbound: SSH (22) from your IP
  - Inbound: HTTP/HTTPS if needed
  - Outbound: All traffic (default)

### Private EC2 Instance

- Subnet: `private-subnet-1` or `private-subnet-2`
- Public IP: ❌ None
- Access via:
  - SSM (attach `AmazonSSMManagedInstanceCore` role)
  - Or SSH via bastion instance in public subnet
- Outbound internet: via NAT Gateway

---

## ✅ Architecture Overview

```VPC: 10.0.0.0/16
├── IGW → public-subnet-[1,2]
│   ├── EC2 (public IP) → Internet
│   └── NAT Gateway
└── private-subnet-[1,2]
    └── EC2 (no public IP) → NAT → IGW → Internet```


---

## 🔐 Notes

- Only public subnet instances can receive traffic **from internet** (via IGW)
- Private subnet instances can **initiate** internet traffic, but can’t be reached from outside
- Use SSM for managing private instances securely

---

**✅ VPC setup is now ready for production use with secure internet exposure and private backend resources.**

