# Screenshots

AWS Console screenshots demonstrating the production-style VPC infrastructure.

## Contents

| File | Description |
|---|---|
| `01-vpc-resource-map.png` | VPC resource map showing subnets, route tables, IGW, and NAT gateways across AZs |
| `02-target-group.png` | Target group configuration — HTTP on port 8000, linked to ALB |
| `03-auto-scaling-group.png` | Auto Scaling Group with launch template, capacity settings, and multi-AZ |
| `04-security-group.png` | Security group inbound rules (ports 8000, 22, 80) |
| `05-ec2-instances.png` | Running EC2 instances across Availability Zones with health checks passed |
