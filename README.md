# 🚀 DevOps Lab: Practical DevOps Projects

This repository contains hands-on DevOps projects that progress from basic to advanced implementations. Learn by solving real problems with Docker, Terraform, and GitHub Actions.

## Project Structure

- **projeto-devops-fase-1**: Containerize and manually deploy a static website to AWS
- **projeto-devops-fase-2**: Automate infrastructure with Terraform (IaC)
- **projeto-devops-fase-3**: Full CI/CD automation with GitHub Actions + Terraform + Docker

Each folder contains a detailed README with step-by-step instructions.

## 📋 Prerequisites

**⚠️ IMPORTANT**: Basic to intermediate knowledge of development and infrastructure required.

### Required Skills:

- **Linux/Unix**: Terminal navigation, file editing, permissions, SSH
- **AWS**: EC2, IAM, VPC, Security Groups, AWS CLI
- **Docker**: Images, containers, Dockerfile, registries (Docker Hub, ECR)
- **Terraform**: IaC concepts, basic commands (init, plan, apply, destroy)
- **Git/GitHub**: Basic commands and repository management

---

## ❓ Why This Approach?

Learn by solving real problems first, then understand the theory. Each project starts with a concrete challenge (like manual deployment chaos) and introduces tools (Docker, Terraform, CI/CD) as solutions. This problem-based learning makes concepts memorable and practical.

## 🎯 Projects Overview

Deploy a static website (HTML/CSS/JS) to AWS with progressively automated DevOps processes.

### Project 1: Docker Containerization & Manual AWS Deploy (Basic)
- **Problem**: "Works on my machine" – dependency conflicts cause deployment failures
- **Solution**: Containerize with Docker, push to ECR, manual deploy to EC2
- **Tools**: Docker, AWS CLI, ECR, EC2, Security Groups
- **Duration**: 2-3 hours

### Project 2: Infrastructure Automation with Terraform (Intermediate)
- **Problem**: Manual AWS console clicks cause inconsistencies and configuration drift
- **Solution**: Infrastructure as Code with Terraform – declare EC2, ECR, IAM resources in HCL
- **Tools**: Terraform (init/plan/apply/destroy), S3 backend for state
- **Duration**: 2-4 hours

### Project 3: Full CI/CD Automation (Advanced)
- **Problem**: Manual deploys create bottlenecks, human errors, and lack of auditability
- **Solution**: GitHub Actions pipelines automate Docker builds, Terraform plans, and deployments
- **Tools**: GitHub Actions (workflows, secrets, approvals), multi-repo integration
- **Duration**: 3-5 hours

## 🔧 Getting Started

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/PedroConceicao13/devops-project.git
   cd devops-project
   ```

2. **Start with Phase 1**: Go to `projeto-devops-fase-1` and follow the README

3. **Setup**: 
   - Create a free AWS account (watch for costs – use Free Tier)
   - Install Docker, Terraform, and AWS CLI
   - Use VS Code for editing

4. **Important**: 
   - Test locally before deploying
   - Clean up AWS resources after completion to avoid charges
   - Replace placeholders (AWS regions, repo names) with your own values

## 🎓 What You'll Learn

- **Containerization** (Docker): Eliminate environment inconsistencies
- **Infrastructure as Code** (Terraform): Automate and version infrastructure
- **CI/CD** (GitHub Actions): Orchestrate fast, secure deployments
- **Best Practices**: Secrets management, approvals, state locking, drift detection

Progress from manual deployments to fully automated CI/CD pipelines.

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Terraform Learning](https://learn.hashicorp.com/terraform)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- Book: "The DevOps Handbook"
- Communities: Reddit r/devops, Stack Overflow

## 📝 Notes

DevOps is about culture and automation. When stuck, search for the error message – this builds real troubleshooting skills!

---

Developed with ❤️ for the DevOps journey
