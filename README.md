# 🏗️ Production-Style AWS VPC Infrastructure Project

A production-style AWS VPC infrastructure built from scratch, demonstrating real-world cloud networking, high availability, load balancing, and auto scaling patterns. This project showcases a secure, multi-AZ architecture with private EC2 instances fronted by an internet-facing Application Load Balancer.

> **Note:** This is a production-**style** project designed for learning and demonstration purposes. It follows production best practices and architectural patterns but is not an actual production environment.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#-architecture)
- [Architecture Diagram](#-architecture-diagram)
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

## 📐 Architecture Diagram

![Architecture Diagram](architecture/architecture.png)

> The architecture diagram illustrates the complete infrastructure layout including VPC boundaries, subnet placement, ALB configuration, and EC2 instance distribution across Availability Zones.

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

