# DevOps Learning Log 📓

Daily notes from my DevOps learning journey — Kubernetes, Terraform, AWS, CI/CD, Linux deep dives.

> "The best way to learn is to build, break, and document."

## 📅 Learning Tracker

| Topic | Status | Notes |
|---|---|---|
| GitLab CI/CD | ✅ Production experience | Used at ESDS for enterprise pipelines |
| Docker | ✅ Production experience | Multi-stage builds, container security |
| Kubernetes basics | ✅ Hands-on | Deployments, Services, ConfigMaps |
| Jenkins | ✅ Production experience | Declarative pipelines, quality gates |
| RHEL/CentOS Admin | ✅ Deep experience | Built custom distro at work |
| Python automation | ✅ Production experience | REST API testing, system scripting |
| Bash scripting | ✅ Production experience | RPM pipelines, sysadmin tooling |
| Azure AZ-900 | ✅ Certified | Core cloud concepts |
| Kubernetes advanced | 🟡 In progress | Helm, ArgoCD, RBAC |
| Terraform | 🟡 In progress | Modules, state, workspaces |
| AWS SAA-C03 | 🟡 Studying | VPC, EKS, Lambda, IAM, S3 |
| GitHub Actions | 🟡 Learning | Workflow YAML, reusable actions |
| Prometheus + Grafana | 📋 Planned | Observability stack |
| ArgoCD / GitOps | 📋 Planned | CD for Kubernetes |
| Ansible | 📋 Planned | Configuration management |

---

## 📂 Notes by Topic

- [`kubernetes/`](./kubernetes/) — K8s concepts, YAML examples, Helm notes
- [`terraform/`](./terraform/) — IaC patterns, module design, state management
- [`aws/`](./aws/) — SAA-C03 study notes, service summaries
- [`cicd/`](./cicd/) — Pipeline patterns, GitLab CI, Jenkins, GitHub Actions
- [`linux/`](./linux/) — RHEL admin, RPM packaging, systemd, Kickstart
- [`python/`](./python/) — Automation scripts, PyTest patterns, API testing

---

## 🗓 Recent Entries

### 2026-03 — Kubernetes Deep Dive
- Studied Pod lifecycle, resource limits, liveness/readiness probes
- Practiced Helm chart creation and values override
- Looked into ArgoCD for GitOps-style CD

### 2026-02 — Terraform Fundamentals  
- Completed modules: `providers`, `variables`, `outputs`, `locals`
- Built a simple AWS VPC with public/private subnets in Terraform
- Learned about `terraform state` and remote backends (S3 + DynamoDB lock)

### 2026-01 — AWS SAA-C03 Study
- Covered: VPC, subnets, route tables, security groups, NACLs
- IAM: policies, roles, instance profiles, cross-account access
- Compute: EC2 types, ASG, ELB (ALB/NLB), Launch Templates
- Storage: S3 classes, lifecycle policies, versioning, replication

### 2025-12 — GitOps & Advanced CI/CD
- Explored ArgoCD concepts: app of apps, sync policies, RBAC
- Built multi-environment GitLab CI pipeline with environment-specific rules
- Studied blue-green and canary deployment strategies

### 2025-11 — Docker Advanced Topics
- Multi-stage builds for smaller images
- Docker security: non-root users, read-only filesystems, image scanning
- Docker Compose for local development stacks

---

*Learning in public. All notes are personal and may contain errors — PRs welcome!*
