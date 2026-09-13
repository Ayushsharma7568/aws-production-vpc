# 🏗️ Production-Style AWS VPC Infrastructure Project

A production-style AWS VPC infrastructure built from scratch, demonstrating real-world cloud networking, high availability, load balancing, and auto scaling patterns. This project showcases a secure, multi-AZ architecture with private EC2 instances fronted by an internet-facing Application Load Balancer.

> **Note:** This is a production-**style** project designed for learning and demonstration purposes. It follows production best practices and architectural patterns but is not an actual production environment.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Traffic Flow](#-traffic-flow)
- [AWS Services Used](#-aws-services-used)
- [VPC and Subnet Design](#-vpc-and-subnet-design)
- [Public vs Private Subnets](#-public-vs-private-subnets)
- [Internet Gateway](#-internet-gateway)
- [Application Load Balancer](#-application-load-balancer)
- [Target Group and Health Checks](#-target-group-and-health-checks)
- [EC2 Instances](#-ec2-instances)
- [Launch Template](#-launch-template)
- [Auto Scaling Group](#-auto-scaling-group)
- [Security Groups](#-security-groups)
- [Testing](#-testing)
- [Troubleshooting](#-troubleshooting)
- [Key Learnings](#-key-learnings)

---

## 🔍 Project Overview

This project implements a **production-style AWS VPC infrastructure** that demonstrates how real-world cloud architectures are designed and deployed. The infrastructure follows AWS Well-Architected Framework principles, emphasizing:

- **High Availability** — Resources distributed across multiple Availability Zones
- **Security** — EC2 instances placed in private subnets, inaccessible directly from the internet
- **Scalability** — Auto Scaling Group dynamically adjusts capacity based on demand
- **Load Balancing** — Application Load Balancer distributes traffic across healthy instances
- **Health Monitoring** — HTTP health checks ensure only healthy instances receive traffic

### What This Project Demonstrates

| Concept | Implementation |
|---|---|
| Network Isolation | Custom VPC with public and private subnets |
| Multi-AZ Deployment | Resources spread across multiple Availability Zones |
| Secure Access | Private instances accessible only through the ALB |
| Load Distribution | ALB with target group and health checks |
| Auto Scaling | Launch template + Auto Scaling Group for elasticity |
| Traffic Control | Security groups acting as virtual firewalls |

---

## 🏛️ Architecture

The infrastructure follows a **hub-and-spoke networking model** with clear separation between public-facing and private resources:

```
┌─────────────────────────────────────────────────────────────────────┐
│                          AWS VPC                                     │
│                                                                      │
│   ┌──────────────────────┐      ┌──────────────────────┐            │
│   │   Public Subnet       │      │   Public Subnet       │           │
│   │   (Availability       │      │   (Availability       │           │
│   │    Zone A)            │      │    Zone B)            │           │
│   │                       │      │                       │           │
│   │  ┌─────────────────┐ │      │  ┌─────────────────┐ │           │
│   │  │      ALB        │ │      │  │      ALB        │ │           │
│   │  │   (Internet-    │ │      │  │   (Internet-    │ │           │
│   │  │    facing)      │ │      │  │    facing)      │ │           │
│   │  └─────────────────┘ │      │  └─────────────────┘ │           │
│   └──────────────────────┘      └──────────────────────┘            │
│                                                                      │
│   ┌──────────────────────┐      ┌──────────────────────┐            │
│   │  Private Subnet       │      │  Private Subnet       │           │
│   │  (Availability        │      │  (Availability        │           │
│   │   Zone A)             │      │   Zone B)             │           │
│   │                       │      │                       │           │
│   │  ┌─────────────────┐ │      │  ┌─────────────────┐ │           │
│   │  │   EC2 Instance  │ │      │  │   EC2 Instance  │ │           │
│   │  │   (Port 8000)   │ │      │  │   (Port 8000)   │ │           │
│   │  └─────────────────┘ │      │  └─────────────────┘ │           │
│   └──────────────────────┘      └──────────────────────┘            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Architectural Decisions

1. **Private instances** — EC2 instances are placed in private subnets with no direct internet access, reducing the attack surface
2. **ALB in public subnets** — The Application Load Balancer spans public subnets, serving as the single entry point
3. **Multi-AZ** — Resources are distributed across multiple Availability Zones for fault tolerance
4. **Port 8000** — Application runs on port 8000, with the ALB handling external HTTP (port 80) traffic

---

## 🔄 Traffic Flow

Understanding how traffic flows through the infrastructure is critical:

```
    🌐 Internet (User Request)
         │
         ▼
    ┌─────────────┐
    │   Internet   │
    │   Gateway    │
    └─────┬───────┘
          │
          ▼
    ┌─────────────────────────────┐
    │   Application Load Balancer  │
    │   (Internet-facing, HTTP:80) │
    │   Lives in PUBLIC subnets    │
    └─────────────┬───────────────┘
                  │
                  ▼
    ┌─────────────────────────────┐
    │       Target Group           │
    │   (Health Check: HTTP:8000)  │
    └──────────┬──────────────────┘
               │
          ┌────┴────┐
          ▼         ▼
    ┌──────────┐  ┌──────────┐
    │   EC2    │  │   EC2    │
    │ Instance │  │ Instance │
    │ (AZ-A)   │  │ (AZ-B)   │
    │ Port 8000│  │ Port 8000│
    │ PRIVATE  │  │ PRIVATE  │
    └──────────┘  └──────────┘
```

### Step-by-Step Traffic Flow

1. **User sends a request** → Hits the ALB's DNS endpoint over HTTP (port 80)
2. **Internet Gateway** → Routes the traffic into the VPC
3. **ALB receives the request** → Evaluates listener rules and forwards to the target group
4. **Target Group** → Selects a healthy EC2 instance using the configured routing algorithm
5. **EC2 instance processes the request** → Application running on port 8000 handles the request
6. **Response flows back** → Through the ALB to the user

> **Important:** Users never communicate directly with the EC2 instances. All traffic is mediated through the ALB, providing a layer of security and abstraction.

---

## ☁️ AWS Services Used

| AWS Service | Purpose | Key Configuration |
|---|---|---|
| **VPC** | Isolated virtual network | Custom CIDR block, DNS enabled |
| **Subnets** | Network segmentation | Public + Private across multiple AZs |
| **Internet Gateway** | Internet connectivity | Attached to VPC, routes public traffic |
| **Application Load Balancer** | Traffic distribution | Internet-facing, HTTP listener on port 80 |
| **Target Group** | Instance grouping | HTTP health checks on port 8000 |
| **EC2 Instances** | Compute resources | Running application on port 8000 |
| **Launch Template** | Instance configuration | AMI, instance type, security group, user data |
| **Auto Scaling Group** | Capacity management | Min/Max/Desired capacity, multi-AZ |
| **Security Groups** | Network firewalls | Inbound/Outbound rules per resource type |

---

## 🌐 VPC and Subnet Design

### VPC (Virtual Private Cloud)

The VPC serves as the foundational networking layer, providing an isolated virtual network within AWS:

- **Custom CIDR Block** — Defines the IP address range for the entire VPC
- **DNS Support** — Enabled for internal DNS resolution
- **DNS Hostnames** — Enabled so EC2 instances receive public DNS names (when applicable)

### Subnet Layout

The infrastructure uses a **multi-AZ subnet architecture** with both public and private subnets:

```
VPC CIDR Block
├── Public Subnet  (AZ-A)  →  ALB, Internet-facing resources
├── Public Subnet  (AZ-B)  →  ALB, Internet-facing resources
├── Private Subnet (AZ-A)  →  EC2 instances (application workloads)
└── Private Subnet (AZ-B)  →  EC2 instances (application workloads)
```

### Why Multiple Availability Zones?

- **Fault tolerance** — If one AZ experiences issues, the other AZ continues serving traffic
- **High availability** — ALB automatically routes traffic to healthy instances in available AZs
- **AWS best practice** — Production workloads should always span at least 2 AZs

---

## 🔒 Public vs Private Subnets

Understanding the distinction between public and private subnets is fundamental to this architecture:

### Public Subnets

| Property | Value |
|---|---|
| **Route to Internet** | Yes — via Internet Gateway |
| **Auto-assign Public IP** | Enabled |
| **Hosts** | Application Load Balancer |
| **Purpose** | Accept inbound internet traffic |

### Private Subnets

| Property | Value |
|---|---|
| **Route to Internet** | No direct route |
| **Auto-assign Public IP** | Disabled |
| **Hosts** | EC2 instances (application servers) |
| **Purpose** | Run application workloads securely |

### Why Place EC2 Instances in Private Subnets?

1. **Reduced attack surface** — Instances have no public IP addresses and cannot be reached directly from the internet
2. **Defense in depth** — Even if a security group rule is misconfigured, the lack of a public route provides an additional layer of protection
3. **Compliance** — Many security frameworks require backend servers to be in private subnets
4. **Controlled access** — All inbound traffic must pass through the ALB, which provides logging, monitoring, and traffic management

---

## 🚪 Internet Gateway

The Internet Gateway (IGW) is a horizontally scaled, redundant, and highly available VPC component that enables communication between the VPC and the internet.

### How It Works in This Architecture

```
Internet  ←→  Internet Gateway  ←→  Public Subnets (ALB)
                                         │
                                    Private Subnets (EC2)
                                    (NO direct internet route)
```

### Key Points

- **Attached to the VPC** — One IGW per VPC
- **Route table association** — Only public subnet route tables have a route to the IGW (`0.0.0.0/0 → IGW`)
- **Private subnets** — Do NOT have a route to the IGW, ensuring instances remain isolated
- **Stateful** — Return traffic for outbound requests is automatically allowed

---

## ⚖️ Application Load Balancer

The Application Load Balancer (ALB) is the **single entry point** for all user traffic into the application.

### Configuration

| Setting | Value |
|---|---|
| **Scheme** | Internet-facing |
| **Type** | Application Load Balancer (Layer 7) |
| **Listener** | HTTP on port 80 |
| **Subnets** | Deployed across public subnets in multiple AZs |
| **Target** | Forwards traffic to Target Group on port 8000 |

### Why ALB?

- **Layer 7 load balancing** — Operates at the application layer (HTTP/HTTPS), enabling content-based routing
- **Health checks** — Automatically removes unhealthy instances from rotation
- **Cross-AZ balancing** — Distributes traffic evenly across instances in multiple Availability Zones
- **Scalability** — AWS manages the ALB's scaling automatically
- **Security** — Acts as a reverse proxy, hiding backend instance details from users

### ALB Listener Rules

```
Listener (HTTP:80)
  └── Default Action: Forward to Target Group (Port 8000)
```

---

## 🎯 Target Group and Health Checks

The Target Group defines the set of EC2 instances that receive traffic from the ALB.

### Target Group Configuration

| Setting | Value |
|---|---|
| **Target Type** | Instance |
| **Protocol** | HTTP |
| **Port** | 8000 |
| **VPC** | Project VPC |

### Health Check Configuration

| Setting | Value |
|---|---|
| **Protocol** | HTTP |
| **Port** | 8000 |
| **Path** | `/` (root path) |
| **Healthy Threshold** | Number of consecutive successes to mark healthy |
| **Unhealthy Threshold** | Number of consecutive failures to mark unhealthy |
| **Timeout** | Seconds to wait for a health check response |
| **Interval** | Seconds between health checks |

### Health Check Flow

```
ALB  ──(HTTP GET /)──►  EC2 Instance (Port 8000)
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              HTTP 200 OK         Non-200 / Timeout
              (Healthy ✅)         (Unhealthy ❌)
```

### Why Health Checks Matter

- **Automatic failover** — Unhealthy instances are removed from the target group; traffic is routed only to healthy instances
- **Zero-downtime deployments** — New instances must pass health checks before receiving traffic
- **Monitoring** — Provides visibility into application health across the fleet

---

## 💻 EC2 Instances

The EC2 instances are the **compute backbone** of the architecture, running the application workload.

### Instance Configuration

| Property | Details |
|---|---|
| **Placement** | Private subnets (no public IP) |
| **Application Port** | 8000 |
| **Multi-AZ** | Distributed across multiple Availability Zones |
| **Access** | Only accessible through the ALB |
| **Management** | Launched and managed by Auto Scaling Group |

### Why Private Instances?

- Instances have **no public IP address**
- Instances have **no direct internet route**
- All inbound traffic **must flow through the ALB**
- This is the **recommended pattern** for production web applications

---

## 📋 Launch Template

The Launch Template defines the **blueprint** for every EC2 instance launched by the Auto Scaling Group.

### What the Launch Template Specifies

| Parameter | Purpose |
|---|---|
| **AMI ID** | The base machine image for instances |
| **Instance Type** | Compute capacity (CPU, memory) |
| **Security Group** | Firewall rules applied to instances |
| **User Data** | Bootstrap script to configure the instance on launch |

### User Data Script

The user data script runs automatically when each instance launches, typically:

1. Updating system packages
2. Installing the application runtime/dependencies
3. Starting the application on port 8000

> **Note:** The user data script ensures every instance launched by the ASG is identically configured and immediately ready to serve traffic.

---

## 📈 Auto Scaling Group

The Auto Scaling Group (ASG) **automates capacity management** by launching and terminating EC2 instances based on demand.

### ASG Configuration

| Setting | Value |
|---|---|
| **Launch Template** | References the project launch template |
| **VPC Subnets** | Private subnets across multiple AZs |
| **Target Group** | Attached to the ALB target group |
| **Min Capacity** | Minimum number of running instances |
| **Max Capacity** | Maximum number of running instances |
| **Desired Capacity** | Target number of instances to maintain |

### How Auto Scaling Works

```
                    ┌───────────────────┐
                    │  Auto Scaling      │
                    │  Group             │
                    └────────┬──────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ Instance │  │ Instance │  │ Instance │
        │ (AZ-A)   │  │ (AZ-B)   │  │ (AZ-A)   │
        └──────────┘  └──────────┘  └──────────┘

    Scales OUT (adds instances) when demand increases
    Scales IN (removes instances) when demand decreases
```

### Key Benefits

- **High availability** — Automatically replaces failed instances
- **Cost optimization** — Scales down during low-traffic periods
- **Consistent capacity** — Maintains desired number of healthy instances
- **AZ balancing** — Distributes instances evenly across Availability Zones
- **Integration** — Automatically registers new instances with the target group

---

## 🛡️ Security Groups

Security Groups act as **virtual firewalls** controlling inbound and outbound traffic for AWS resources. This architecture uses separate security groups for the ALB and EC2 instances, following the principle of least privilege.

### ALB Security Group

Controls traffic to and from the Application Load Balancer:

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| HTTP | TCP | 80 | `0.0.0.0/0` (Anywhere) | Allow internet traffic to ALB |

**Outbound Rules:**

| Type | Protocol | Port | Destination | Purpose |
|---|---|---|---|---|
| Custom TCP | TCP | 8000 | EC2 Security Group | Allow ALB to reach EC2 instances |

### EC2 Instance Security Group

Controls traffic to and from the private EC2 instances:

**Inbound Rules:**

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| Custom TCP | TCP | 8000 | ALB Security Group | Allow traffic ONLY from the ALB |

**Outbound Rules:**

| Type | Protocol | Port | Destination | Purpose |
|---|---|---|---|---|
| All Traffic | All | All | `0.0.0.0/0` | Allow outbound traffic (updates, etc.) |

### Security Group Chain

```
Internet (HTTP:80)
    │
    ▼
┌─────────────────────────┐
│  ALB Security Group      │
│  Inbound:  HTTP:80 from  │
│            0.0.0.0/0     │
│  Outbound: TCP:8000 to   │
│            EC2 SG        │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│  EC2 Security Group      │
│  Inbound:  TCP:8000 from │
│            ALB SG only   │
│  Outbound: All traffic   │
└─────────────────────────┘
```

### Key Security Design Decisions

1. **SG chaining** — The EC2 security group references the ALB security group as its source, not an IP range. This means only traffic originating from the ALB is permitted.
2. **No SSH from internet** — There is no inbound SSH (port 22) rule from `0.0.0.0/0`, preventing direct remote access from the internet.
3. **Minimal port exposure** — Only port 8000 is open on EC2 instances, and only from the ALB.
4. **Principle of least privilege** — Each security group allows only the minimum required traffic.

---

## 🧪 Testing

### Verifying the Infrastructure

#### 1. ALB DNS Access

Test that the application is reachable through the ALB:

```bash
# Get the ALB DNS name from the AWS Console or CLI
# Then access it in a browser or via curl:
curl http://<ALB-DNS-Name>
```

- ✅ **Expected:** HTTP 200 response with application content
- ❌ **If failing:** Check security groups, target group health, and listener configuration

#### 2. Target Group Health Checks

Verify that EC2 instances are passing health checks:

1. Navigate to **EC2 → Target Groups** in the AWS Console
2. Select the target group
3. Check the **Targets** tab
4. All registered instances should show status: **healthy**

| Status | Meaning |
|---|---|
| `healthy` | Instance is passing health checks ✅ |
| `unhealthy` | Instance is failing health checks ❌ |
| `draining` | Instance is being deregistered ⚠️ |
| `initial` | Health checks are in progress 🔄 |

#### 3. Multi-AZ Verification

Confirm instances are distributed across Availability Zones:

1. Navigate to **EC2 → Instances** in the AWS Console
2. Check the **Availability Zone** column for each instance
3. Instances should be spread across at least 2 different AZs

#### 4. Security Group Validation

Verify that EC2 instances are NOT directly accessible from the internet:

```bash
# This should NOT work (instance has no public IP and no direct route)
curl http://<Private-IP>:8000    # Should timeout or fail

# This SHOULD work (traffic goes through ALB)
curl http://<ALB-DNS-Name>       # Should return application response
```

#### 5. Auto Scaling Verification

Confirm the Auto Scaling Group is maintaining desired capacity:

1. Navigate to **EC2 → Auto Scaling Groups** in the AWS Console
2. Verify **Desired**, **Min**, and **Max** capacity values
3. Check the **Activity** tab for recent scaling events
4. Confirm instances match the desired count

---

## 🔧 Troubleshooting

### Common Issues and Solutions

#### ❌ ALB returns 502 Bad Gateway

| Possible Cause | Solution |
|---|---|
| Application not running on EC2 | SSH into instance and verify the app is running on port 8000 |
| Wrong port in target group | Ensure target group port matches the application port (8000) |
| Security group blocking traffic | Verify EC2 SG allows inbound TCP:8000 from ALB SG |
| Health checks failing | Check health check path and port configuration |

#### ❌ All targets showing "unhealthy"

1. **Check the application** — Is the app actually running on port 8000?
   ```bash
   # On the EC2 instance:
   curl http://localhost:8000
   ```
2. **Check the health check path** — Does the configured path return HTTP 200?
3. **Check security groups** — Does the EC2 SG allow traffic from the ALB SG on port 8000?
4. **Check the health check port** — Is it set to 8000 (not 80)?

#### ❌ Cannot access ALB DNS from browser

1. **Check ALB security group** — Inbound rule must allow HTTP (port 80) from `0.0.0.0/0`
2. **Check ALB scheme** — Must be "internet-facing" (not "internal")
3. **Check ALB subnets** — Must be in public subnets with Internet Gateway route
4. **DNS propagation** — Wait a few minutes for DNS to propagate after ALB creation

#### ❌ Auto Scaling Group not launching instances

1. **Check launch template** — Verify AMI ID is valid in the selected region
2. **Check subnet configuration** — ASG subnets must exist and have available IP addresses
3. **Check service limits** — Ensure you haven't hit EC2 instance limits
4. **Check the Activity tab** — Look for error messages in scaling activities

#### ❌ Instances launch but immediately terminate

1. **Check user data script** — Errors in the bootstrap script can cause instance failures
2. **Check instance logs** — Review `/var/log/cloud-init-output.log` on the instance
3. **Check health checks** — If the target group health check fails repeatedly, the ASG may replace instances

### Debugging Checklist

```
□ VPC has an Internet Gateway attached
□ Public subnets have a route to the IGW (0.0.0.0/0 → IGW)
□ ALB is in public subnets
□ ALB security group allows inbound HTTP:80
□ ALB listener forwards to the correct target group
□ Target group port is 8000
□ Target group health check port is 8000
□ EC2 security group allows inbound TCP:8000 from ALB SG
□ Application is running and listening on port 8000
□ EC2 instances are in private subnets
□ Auto Scaling Group references correct launch template
□ Auto Scaling Group is attached to the target group
```

---

## 📚 Key Learnings

### 1. Network Architecture

- A **VPC** provides complete network isolation — you control the IP ranges, subnets, routing, and gateways
- **Subnets** determine whether resources are publicly accessible (public subnet with IGW route) or isolated (private subnet without IGW route)
- **Availability Zones** are independent failure domains — deploying across multiple AZs is essential for high availability

### 2. Security in Depth

- **Private subnets** are the first line of defense — instances without public IPs cannot be directly reached
- **Security group chaining** (referencing SG IDs instead of IP ranges) creates a trust relationship between resources
- The principle of **least privilege** means opening only the ports that are absolutely necessary
- **Never expose backend instances directly** — always use a load balancer as a reverse proxy

### 3. Load Balancing and Health

- An **ALB** operates at Layer 7 (HTTP) and can make routing decisions based on content
- **Health checks** are critical — they ensure the ALB only sends traffic to instances that can actually serve requests
- **Target groups** decouple the ALB from specific instances, enabling dynamic scaling

### 4. Auto Scaling

- **Launch templates** ensure consistency — every instance is identically configured
- **Auto Scaling Groups** automate capacity management, replacing failed instances and scaling with demand
- **User data scripts** enable zero-touch instance provisioning — instances are production-ready from the moment they launch

### 5. Infrastructure Thinking

- Always start with a clear **architecture diagram** before building
- Think about **traffic flow** — how does a request get from the user to the application and back?
- **Security groups** are stateful firewalls — return traffic is automatically allowed
- Every AWS service has a specific role — understanding **why** each service is needed is more important than knowing **how** to configure it

### 6. Operational Best Practices

- **Never commit credentials** — Use `.gitignore` to prevent accidental commits of `.pem` files, `.env` files, and AWS credentials
- **Use private subnets** for backend workloads — this is an industry standard, not an optional best practice
- **Monitor health checks** — They are the pulse of your application; if health checks fail, users are impacted
- **Document everything** — Future you (and your team) will thank you for clear architecture documentation

---

## 📝 License

This project is for educational and demonstration purposes.

---

## 🙏 Acknowledgments

Built as a hands-on project to learn and demonstrate AWS VPC infrastructure, networking, and cloud architecture best practices.
