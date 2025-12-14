# AWS Deployment with CI/CD Guide

A comprehensive guide covering different deployment approaches on AWS - from basic manual setup to enterprise-grade CI/CD pipelines. Perfect for learning deployment architecture and best practices.

---

## 📋 Table of Contents

1. [Deployment Journey Overview](#deployment-journey-overview)
2. [Deployment Approaches](#deployment-approaches)
3. [Architecture Comparison](#architecture-comparison)
4. [Quick Start Guide](#quick-start-guide)
5. [Learning Path](#learning-path)

---

## 🚀 Deployment Journey Overview

Deployment maturity grows through different stages. This guide covers the complete spectrum:

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT MATURITY LEVELS                   │
└─────────────────────────────────────────────────────────────────┘

LEVEL 1: BEGINNER                                      LEVEL 4: ENTERPRISE
│                                                                  │
▼                                                                  ▼

┌──────────────────┐   ┌──────────────────┐   ┌───────────────────┐
│   Basic Setup    │   │   Docker Setup   │   │  Enterprise CI/CD │
│  (Manual + SSH)  │   │  (Containerized) │   │  (Full Automated) │
├──────────────────┤   ├──────────────────┤   ├───────────────────┤
│ • SSH Connect    │   │ • Docker Images  │   │ • GitHub Actions  │
│ • Manual Deploy  │   │ • ECR Registry   │   │ • Auto Testing    │
│ • PM2 Process    │   │ • Basic CI/CD    │   │ • Security Scan   │
│ • Nginx Config   │   │ • EC2 Runner     │   │ • Load Balance    │
│ • Low Infra Cost │   │ • Better Scaling │   │ • High Reliability│
│ • Max Manual     │   │ • Moderate Auto  │   │ • Full Auto       │
└──────────────────┘   └──────────────────┘   └───────────────────┘
```

---

## 📊 Deployment Approaches Breakdown

### Approach 1️⃣: Basic PM2 + SSH Deployment

**Use Case:** Learning, small projects, solo development

```
                    YOUR LOCAL MACHINE
                            │
                            │ (SSH Connection)
                            ▼
        ┌──────────────────────────────────┐
        │      AWS EC2 Instance            │
        │  ┌───────────────────────────┐   │
        │  │  • Node.js + npm          │   │
        │  │  • PM2 (Process Manager)  │   │
        │  │  • Nginx (Reverse Proxy)  │   │
        │  └───────────────────────────┘   │
        │           ▲                      |
        │           │ Manual pull          │
        │           │ & start              │
        │    GitHub Repository             │
        └──────────────────────────────────┘
```

**Workflow:**

```bash
1. SSH into EC2
2. Clone/Pull code from GitHub
3. Run npm install
4. Start with PM2
5. Configure Nginx as reverse proxy
```

**Pros:**

- ✅ Simple to understand
- ✅ Low AWS costs
- ✅ Good for learning
- ✅ Full control over process

**Cons:**

- ❌ Manual deployment every time
- ❌ No auto-restart on failure
- ❌ Difficult to scale
- ❌ Zero automation

**Cost:** ~$5-15/month

---

### Approach 2️⃣: Enterprise Docker + ECR + CI/CD Deployment

**Use Case:** Production applications, teams, continuous delivery

```
                ┌────────────────────┐
                │  GitHub Repository │
                └────────┬───────────┘
                         │
                         ▼
            ┌────────────────────────┐
            │ GitHub Actions Workflow│
            │  • Build Docker Image  │
            │  • Run Tests           │
            │  • Push to ECR         │
            └────────────┬───────────┘
                         │
                         ▼
                ┌──────────────────┐
                │  AWS ECR         │
                │  (Image Registry)│
                └────────┬─────────┘
                         │
                         ▼
        ┌────────────────────────────────┐
        │      AWS EC2 Instance          │
        │  ┌──────────────────────────┐  │
        │  │ Docker Container         │  │
        │  │ • Node.js App            │  │
        │  │ • Pull from ECR          │  │
        │  │ • Auto Deploy            │  │
        │  └──────────────────────────┘  │
        └────────────────────────────────┘
```

**Workflow:**

```
Code Push to GitHub
    ↓
GitHub Actions Triggered
    ↓
Run Tests + Build Docker Image
    ↓
Push Image to AWS ECR
    ↓
SSH to EC2 + Pull Latest Image
    ↓
Stop Old Container + Start New
    ↓
Zero Downtime Deployment ✨
```

**Pros:**

- ✅ Fully automated CI/CD
- ✅ Easy scaling with containers
- ✅ Consistent environments
- ✅ Better security
- ✅ Production-grade setup
- ✅ Easy rollback

**Cons:**

- ❌ More complex setup
- ❌ Need to understand Docker
- ❌ Slightly higher costs

**Cost:** ~$15-50/month

---

## 🏢 Enterprise Patterns (What Companies Use)

### Pattern 1: Multi-Stage Deployment

```
DEVELOPMENT         STAGING           PRODUCTION
     │                  │                  │
     ▼                  ▼                  ▼
┌─────────┐        ┌─────────┐       ┌─────────┐
│ Dev EC2 │───────▶│ Stag EC2│──────▶│ Prod EC2│
│  (Auto) │        │  (Auto) │       │(Auto)   │
└─────────┘        └─────────┘       └─────────┘
     ▲                  ▲                  ▲
     └──────────────────┴──────────────────┘
            GitHub Actions Workflow
```

### Pattern 2: Load Balancing + Auto Scaling

```
                    ┌──────────────┐
                    │ Load Balancer│
                    │  (AWS ALB)   │
                    └──────┬───────┘
                  ┌────────┼────────┐
                  ▼        ▼        ▼
            ┌─────────┬─────────┬─────────┐
            │ App-1   │ App-2   │ App-3   │
            │ Docker  │ Docker  │ Docker  │
            │ Container│Container│Container│
            └─────────┴─────────┴─────────┘
                 Managed by Auto-Scaling Group
```

### Pattern 3: Database + Caching Layer

```
     ┌─────────────────────────────────┐
     │   Application Servers (EC2)     │
     │   With Docker Containers        │
     └──────────┬──────────┬───────────┘
                │          │
                ▼          ▼
        ┌──────────────┬────────────┐
        │ AWS RDS      │ AWS ElastiC│
        │ (Database)   │ Cache(Redis)│
        └──────────────┴────────────┘
```

---

## 📚 Architecture Comparison Table

| Feature            | Basic PM2  | Docker + CI/CD | Enterprise        |
| ------------------ | ---------- | -------------- | ----------------- |
| **Setup Time**     | 30 min     | 2-3 hours      | Custom            |
| **Deployment**     | Manual     | Automated      | Fully Automated   |
| **Scalability**    | Poor       | Good           | Excellent         |
| **Reliability**    | Medium     | High           | Very High         |
| **Cost**           | $          | $$             | $$$               |
| **Learning Curve** | Easy       | Medium         | Hard              |
| **Team Size**      | Solo/Small | Small/Medium   | Medium/Large      |
| **Downtime**       | Yes        | Minimal        | Zero (Blue-Green) |
| **Monitoring**     | None       | Basic          | Advanced          |
| **Auto-Rollback**  | Manual     | Manual         | Automatic         |

---

## ⚡ Quick Start Guide

### Option 1: Start with Basic (Recommended for Learning)

```bash
# Follow this path if you're new to deployment
1. Read: 01-basic-pm2-ssh-deployment/README.md
2. Time: ~1 hour
3. Cost: Minimal
4. Next: Learn Docker concepts
```

### Option 2: Go Full Enterprise (Recommended for Production)

```bash
# Follow this path if you need production-ready setup
1. Read: 02-enterprise-docker-cicd-deployment/README.md
2. Time: ~2-3 hours
3. Cost: Moderate
4. Result: Fully automated CI/CD pipeline
```

---

## 🎓 Learning Path

### Phase 1: Understand Basics (Week 1)

```
□ Understand EC2, Security Groups, SSH
□ Learn how Nginx reverse proxy works
□ Deploy basic Node.js app manually
□ Master PM2 process management
→ Checkpoint: Deploy something manually via SSH
```

### Phase 2: Containerization (Week 2)

```
□ Learn Docker fundamentals
□ Understand Dockerfile and layers
□ Learn Docker Compose
□ Create Docker image for your app
→ Checkpoint: Run app in Docker container locally
```

### Phase 3: AWS Container Services (Week 3)

```
□ Understand AWS ECR
□ Learn IAM roles and permissions
□ Practice pushing images to ECR
□ Pull and run images on EC2
→ Checkpoint: Deploy Docker image from ECR to EC2
```

### Phase 4: CI/CD Automation (Week 4)

```
□ Learn GitHub Actions
□ Write workflow files
□ Automate build process
□ Automate push to ECR
□ Automate deployment to EC2
→ Checkpoint: One-command deployment to production
```

### Phase 5: Production Patterns (Week 5+)

```
□ Learn about load balancing
□ Understand auto-scaling
□ Database and cache patterns
□ Security best practices
□ Monitoring and logging
→ Checkpoint: Enterprise-grade deployment
```

---

## 🔧 Tech Stack by Approach

### Basic Deployment Stack

```
Backend: Node.js + Express
Process Manager: PM2
Web Server: Nginx
Infra: AWS EC2
Version Control: Git/GitHub
Deployment: Manual SSH
```

### Enterprise Stack

```
Backend: Node.js + Express
Containerization: Docker
Container Registry: AWS ECR
Orchestration: Docker Compose
CI/CD: GitHub Actions
Web Server: Nginx
Infra: AWS EC2
Load Balancer: AWS ALB (optional)
Database: AWS RDS (optional)
Cache: AWS ElastiCache (optional)
Monitoring: CloudWatch (optional)
```

---

## 📁 Repository Structure

```
AWS-Deployment-with-CICD-Guide/
│
├── README.md (You are here)
│
├── 01-basic-pm2-ssh-deployment/
│   ├── README.md (Step-by-step manual deployment guide)
│   └── (Configuration files & scripts)
│
├── 02-enterprise-docker-cicd-deployment/
│   ├── README.md (Complete CI/CD setup guide)
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── .github/workflows/
│   │   └── deploy.yml (GitHub Actions workflow)
│   └── (Sample app files)
│
└── .git/
```

---

## 🎯 Common Use Cases & Recommendations

### Scenario 1: Learning to Deploy (Solo Developer)

```
START HERE → 01-basic-pm2-ssh-deployment
WHY: Simple, quick, no unnecessary complexity
TIME: 1 hour to deploy
NEXT: Learn Docker after mastering basics
```

### Scenario 2: Building Production App (Startup/SME)

```
START HERE → 02-enterprise-docker-cicd-deployment
WHY: Automated, scalable, production-ready
TIME: 2-3 hours setup, then fully automated
NEXT: Add monitoring and alerts after going live
```

### Scenario 3: Enterprise Application (Large Team)

```
COMBINE BOTH + ADD:
  • Load Balancing (AWS ALB)
  • Database (AWS RDS)
  • Caching (AWS ElastiCache)
  • Monitoring (CloudWatch)
  • Blue-Green Deployments
  • Infrastructure as Code (Terraform/CloudFormation)
```

---

## 💡 Key Concepts to Understand

### Deployment

The process of moving code from development to production

### CI/CD Pipeline

- **CI (Continuous Integration):** Automatically build and test code when pushed
- **CD (Continuous Deployment):** Automatically deploy tested code to production

### Containerization

Package your app with all dependencies in a Docker container

### Infrastructure as Code

Define and manage infrastructure using configuration files

### Load Balancing

Distribute traffic across multiple servers for better reliability

### Auto-Scaling

Automatically increase/decrease servers based on demand

---

## 🚨 Important Security Notes

- ✅ Always use `.env` files for sensitive data
- ✅ Don't commit `.pem` files to Git
- ✅ Use IAM roles instead of access keys
- ✅ Enable Security Groups properly
- ✅ Use HTTPS in production
- ✅ Regularly update packages
- ✅ Use secrets management for CI/CD

---

## 📞 Troubleshooting Quick Links

| Issue                  | Solution                                       |
| ---------------------- | ---------------------------------------------- |
| Can't SSH to EC2       | Check Security Groups allow SSH (port 22)      |
| Port already in use    | Kill process: `lsof -i :PORT` or `kill -9 PID` |
| PM2 not starting       | Check logs: `pm2 logs`                         |
| Docker build fails     | Check Dockerfile syntax and dependencies       |
| GitHub Actions timeout | Increase timeout in workflow file              |
| ECR push fails         | Check IAM permissions                          |

---

## 🎓 Next Steps

1. **Choose Your Path:**

   - Learning? → Start with [Approach 1️⃣](#approach-1️⃣-basic-pm2--ssh-deployment)
   - Production Ready? → Go with [Approach 2️⃣](#approach-2️⃣-enterprise-docker--ecr--cicd-deployment)

2. **Follow the Detailed Guide:**

   - Open the respective folder's README.md
   - Follow step-by-step instructions
   - Test each step before moving forward

3. **Deploy Your Own Project:**
   - Adapt the examples to your codebase
   - Test in development first
   - Deploy to staging
   - Go live with confidence

---

## 📖 Resources

### AWS Documentation

- [EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [ECR Documentation](https://docs.aws.amazon.com/ecr/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Tools Used

- [PM2 Process Manager](https://pm2.keymetrics.io/)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [GitHub Actions](https://docs.github.com/en/actions)

### Learning Resources

- AWS Free Tier for practice
- Docker & Kubernetes fundamentals
- CI/CD concepts
- System design patterns

---

## 📝 License

This guide is open source and available for learning purposes.

---

## ⭐ Did this help?

If this guide helped you understand deployment, please give it a star! 🌟

Share your deployment journey and ask questions in the issues section.

---

**Last Updated:** December 2025  
**Difficulty Level:** Beginner → Advanced  
**Time to Complete:** 4-6 weeks for full mastery
