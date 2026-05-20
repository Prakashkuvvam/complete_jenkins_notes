---
title: "Ch 1: Introduction to CI/CD & Jenkins"
weight: 1
---

# Introduction to CI/CD & Jenkins

## Learning Objectives

After reading this chapter, you will be able to:
- Explain the problems that CI/CD solves in software development
- Describe the stages of a CI/CD pipeline
- Understand what Jenkins is and where it fits in the DevOps toolchain
- Set up Jenkins locally using Docker
- Navigate the Jenkins web interface

---

## 1.1 What is Continuous Integration?

**Continuous Integration (CI)** is a development practice where developers integrate code into a shared repository frequently — ideally multiple times per day. Each integration is verified by an automated build and test process, allowing teams to detect problems early.

### The Problem CI Solves

Before CI, development workflows looked like this:

```
Developer A works on feature  →  merges after weeks
Developer B works on feature  →  merges after weeks
    ↓
Merge hell! Conflicts everywhere. Build breaks. Days lost debugging.
```

With CI:

```
Developer commits small changes → Automated build & test → Fast feedback
Developer commits small changes → Automated build & test → Fast feedback
    ↓
Conflicts are rare. Bugs are caught immediately. Code is always deployable.
```

### Core CI Practices

| Practice | Description |
|----------|-------------|
| **Frequent commits** | Push code to shared repo multiple times daily |
| **Automated builds** | Every commit triggers an automated build |
| **Automated tests** | Unit, integration, and acceptance tests run automatically |
| **Fast feedback** | Developers know within minutes if their changes broke something |
| **Fix broken builds immediately** | The #1 rule: broken builds are the team's top priority |

---

## 1.2 What is Continuous Delivery/Deployment?

**Continuous Delivery (CD)** extends CI by ensuring that the software is always in a deployable state. The build artifact is automatically validated and ready for deployment to production at any time.

**Continuous Deployment** goes one step further — every change that passes the automated pipeline is automatically deployed to production without human intervention.

```
CI:   Commit → Build → Test
CDel: Commit → Build → Test → Stage → Ready to Deploy
CDepl: Commit → Build → Test → Stage → Deploy to Production (automatic)
```

---

## 1.3 The CI/CD Pipeline

A typical CI/CD pipeline consists of several stages:

```groovy
// Conceptual pipeline stages
Pipeline:
  1. Source     ← Fetch code from version control
  2. Build      ← Compile code, resolve dependencies
  3. Test       ← Run unit, integration, and quality checks
  4. Package    ← Create deployable artifact (JAR, Docker image, etc.)
  5. Deploy     ← Push to staging/production environments
```

Each stage provides fast feedback. If tests fail, the pipeline stops immediately.

---

## 1.4 What is Jenkins?

**Jenkins** is an open-source automation server that enables developers to build, test, and deploy software reliably. It is the most widely used CI/CD tool in the industry.

### Key Features

| Feature | Description |
|---------|-------------|
| **Extensible** | 1,800+ plugins for every tool imaginable |
| **Pipeline as Code** | Define build pipelines in a Jenkinsfile (Groovy DSL) |
| **Master/Agent Architecture** | Distribute builds across multiple machines |
| **Built-in Security** | Credential management, RBAC, audit trails |
| **REST API** | Integrate Jenkins with any tool via HTTP API |
| **Multi-platform** | Runs on Windows, Linux, macOS |

### Jenkins vs Other CI/CD Tools

| Feature | Jenkins | GitHub Actions | GitLab CI | CircleCI |
|---------|---------|----------------|-----------|----------|
| **Hosting** | Self-hosted or Cloud | Cloud-only | Hybrid | Cloud-only |
| **Configuration** | Jenkinsfile (Groovy) | YAML | YAML | YAML |
| **Plugin ecosystem** | 1,800+ plugins | Marketplace | Built-in | Built-in |
| **Infrastructure** | Master/agent | Runners | Runners | Docker |
| **Pricing** | Free (open source) | Free tier | Free tier | Free tier |
| **Flexibility** | Maximum | Moderate | Moderate | Moderate |

### When to Choose Jenkins

- You need **self-hosted** control over your CI/CD infrastructure
- Your toolchain includes **niche or legacy tools** with Jenkins plugins
- You need **complex, custom pipeline logic** beyond simple YAML
- You're operating in **air-gapped or security-sensitive** environments

---

## 1.5 Installing Jenkins

### Option 1: Docker (Recommended)

```bash
# Pull and run Jenkins LTS
docker run \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Get the initial admin password
docker logs jenkins 2>&1 | grep "Password"
```

### Option 2: Native Installation (Ubuntu/Debian)

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | \
  sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y fontconfig openjdk-17-jre jenkins
sudo systemctl enable --now jenkins
```

### Option 3: macOS (Homebrew)

```bash
brew install jenkins-lts
brew services start jenkins-lts
```

### Initial Setup

1. Open `http://localhost:8080`
2. Unlock with the initial admin password
3. Install suggested plugins
4. Create the first admin user
5. Set the Jenkins URL

---

## 1.6 Navigating the Jenkins UI

Once logged in, you'll see:

- **Dashboard** — Main landing page with job status and build history
- **New Item** — Create jobs, pipelines, folders, etc.
- **People** — User management
- **Build History** — Recent builds across all jobs
- **Manage Jenkins** — System configuration, plugin management, security
- **My Views** — Custom filtered views of your jobs

---

## Key Takeaways

- CI/CD automates the software delivery process, catching bugs early
- Jenkins is a self-hosted, extensible automation server with 1,800+ plugins
- The Jenkinsfile allows you to define pipelines as code
- Docker is the quickest way to get started with Jenkins locally
- Jenkins's master/agent architecture scales from a single machine to thousands of nodes

---

## Next Steps

→ Continue to [Chapter 2: Jenkins Architecture & Installation]({{< relref "02-jenkins-architecture-and-installation" >}})

→ Or jump to the [Progress Dashboard]({{< relref "25-progress-dashboard" >}}) to track your learning
