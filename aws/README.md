# AWS SAA-C03 Study Notes

## Core Services Summary

### Compute

| Service | Use Case |
|---|---|
| EC2 | Virtual machines — full control |
| Lambda | Serverless functions, event-driven |
| ECS | Managed Docker containers |
| EKS | Managed Kubernetes |
| Fargate | Serverless containers (no EC2 mgmt) |
| Elastic Beanstalk | PaaS — deploy apps without infra mgmt |

### Storage

| Service | Use Case |
|---|---|
| S3 | Object storage — unlimited, durable |
| EBS | Block storage — attached to EC2 |
| EFS | Elastic NFS — shared filesystem |
| S3 Glacier | Archive storage — low cost |
| FSx | Managed Windows/Lustre file systems |

### Networking

| Service | Use Case |
|---|---|
| VPC | Isolated virtual network |
| Route 53 | DNS + health checks + routing policies |
| CloudFront | CDN — global edge delivery |
| ALB | Layer 7 load balancer (HTTP/HTTPS) |
| NLB | Layer 4 load balancer (TCP/UDP) |
| Direct Connect | Dedicated network to AWS |
| VPN Gateway | Site-to-site VPN over internet |

### Databases

| Service | Use Case |
|---|---|
| RDS | Managed relational DB (MySQL, Postgres, etc.) |
| Aurora | AWS-optimized MySQL/Postgres — 5x faster |
| DynamoDB | Managed NoSQL — serverless, single-digit ms |
| ElastiCache | In-memory cache (Redis/Memcached) |
| Redshift | Data warehouse — analytics |

## IAM Key Concepts

```
Users      → Human identities
Groups     → Collection of users
Roles      → Identities for AWS services or cross-account
Policies   → JSON documents defining permissions

Best practices:
- Never use root account for daily tasks
- Enable MFA on all users
- Use roles for EC2/Lambda (not access keys)
- Principle of least privilege
- Use IAM Access Analyzer to find overly permissive policies
```

## VPC Architecture

```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24)
│   ├── Internet Gateway → public traffic
│   ├── NAT Gateway → for private subnet egress
│   └── EC2 (web servers, ALB)
└── Private Subnet (10.0.2.0/24)
    ├── EC2 (app servers, RDS)
    └── Routes via NAT Gateway for outbound
```

**Security Layers:**
- **Security Groups** — stateful, instance-level, allow rules only
- **NACLs** — stateless, subnet-level, allow + deny rules

## S3 Storage Classes

| Class | Use Case | Retrieval |
|---|---|---|
| Standard | Frequent access | Immediate |
| Intelligent-Tiering | Unknown patterns | Immediate |
| Standard-IA | Infrequent access | Immediate |
| One Zone-IA | Non-critical, infrequent | Immediate |
| Glacier Instant | Archive, ms retrieval | Milliseconds |
| Glacier Flexible | Archive | Minutes-hours |
| Glacier Deep Archive | Long-term archive | Hours |

## High Availability Patterns

```
Multi-AZ RDS     → standby in different AZ, auto failover
Read Replicas    → scale reads, cross-region possible
ASG + ALB        → auto scale EC2 across AZs
S3 replication   → cross-region for disaster recovery
Route53 failover → health check-based DNS failover
```

## SAA-C03 Exam Tips

- Know when to use ALB vs NLB vs CLB
- Understand S3 storage class costs and retrieval times
- IAM roles > access keys always
- VPC design: public/private subnets, NAT, IGW
- RDS Multi-AZ = HA, Read Replicas = performance
- Lambda + API Gateway = serverless API
- CloudFront + S3 = static website with global CDN
