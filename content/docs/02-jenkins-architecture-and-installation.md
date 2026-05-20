---
title: "Ch 2: Jenkins Architecture & Installation"
weight: 2
---

# Jenkins Architecture & Installation

## Learning Objectives

After reading this chapter, you will be able to:
- Explain the Jenkins master/agent architecture
- Understand the different types of nodes and their roles
- Deploy Jenkins across various platforms
- Configure nodes for distributed builds
- Understand Jenkins' system requirements and scaling model

---

## 2.1 The Master/Agent Architecture

Jenkins follows a **master/agent** (also called controller/agent) architecture:

```
┌─────────────────────────────────────────────────────────┐
│                     Jenkins Master                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │   Scheduler  │  │  Web UI     │  │  REST API   │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  Pipeline   │  │  Plugin Mgr │  │  Credentials │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
└────────────────────────┬────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
  ┌────────────┐  ┌────────────┐  ┌────────────┐
  │  Agent 1   │  │  Agent 2   │  │  Agent 3   │
  │ (Linux)    │  │ (Windows)  │  │ (macOS)    │
  └────────────┘  └────────────┘  └────────────┘
```

### Master (Controller)

The master is the central Jenkins server that:
- Serves the web UI and REST API
- Schedules and monitors build jobs
- Stores configuration and build history
- Manages credentials and permissions
- Orchestrates agent connections

> **Important:** In modern Jenkins, the master should NOT run build jobs itself. It should only coordinate.

### Agents (Nodes)

Agents are worker machines that execute build jobs. They:
- Connect to the master via JNLP (TCP) or SSH
- Run the actual build steps (compile, test, package)
- Can be ephemeral (spun up on demand in cloud environments)
- Can run different operating systems than the master

---

## 2.2 System Requirements

### Master (Controller)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 2 cores | 4+ cores |
| **RAM** | 4 GB | 8 GB+ |
| **Disk** | 50 GB | 100 GB+ SSD |
| **Java** | Java 11 | Java 17 |

### Agent

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **CPU** | 1 core | 2+ cores |
| **RAM** | 1 GB | 4 GB+ |
| **Disk** | 10 GB | 50 GB+ |
| **Java** | Java 11 | Java 17 |

---

## 2.3 Installation Methods

### Docker Compose (Production-Like Setup)

```yaml
# docker-compose.yml
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
      - JENKINS_ADMIN_ID=admin
      - JENKINS_ADMIN_PASSWORD=admin

volumes:
  jenkins_home:
```

### WAR File

```bash
# Download and run Jenkins WAR directly
wget https://get.jenkins.io/war-stable/latest/jenkins.war
java -jar jenkins.war --httpPort=8080
```

### Kubernetes (Helm)

```bash
# Add Jenkins Helm repo
helm repo add jenkins https://charts.jenkins.io
helm repo update

# Install Jenkins
helm upgrade --install jenkins jenkins/jenkins \
  --set controller.serviceType=LoadBalancer
```

---

## 2.4 Configuring Agents

### Adding a Permanent Agent (SSH)

1. Go to **Dashboard → Manage Jenkins → Nodes → New Node**
2. Set node name and select "Permanent Agent"
3. Configure:

```
Name: linux-builder-01
Remote root directory: /home/jenkins/agent
Labels: linux amd64 docker
Launch method: Launch agents via SSH
Host: 192.168.1.100
Credentials: (SSH key or user/password)
Host Key Verification Strategy: Known hosts file
Availability: Keep this agent online as much as possible
```

### Adding a Permanent Agent (JNLP)

1. Go to **Dashboard → Manage Jenkins → Nodes → New Node**
2. Select "Permanent Agent", set launch method to "Launch agent by connecting it to the controller"
3. On the agent machine:

```bash
# Download agent.jar
curl -O http://jenkins-master:8080/jnlpJars/agent.jar

# Connect to master
java -jar agent.jar \
  -url http://jenkins-master:8080 \
  -secret <SECRET> \
  -name "agent-01" \
  -workDir "/home/jenkins/agent"
```

---

## 2.5 Using Agents in Pipelines

```groovy
pipeline {
    agent none  // No global agent — assign per stage
    
    stages {
        stage('Build on Linux') {
            agent { label 'linux' }
            steps {
                sh 'make build'
            }
        }
        
        stage('Test on Windows') {
            agent { label 'windows' }
            steps {
                bat 'mvn test'
            }
        }
        
        stage('Package') {
            agent { label 'docker' }
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }
    }
}
```

### Agent Labels

Labels help you route jobs to the right nodes:

```groovy
agent { label 'linux && docker' }      // Linux machine with Docker
agent { label 'high-mem || gpu' }      // High-memory or GPU machine
agent { label '!spot-instance' }       // Exclude spot instances
```

---

## 2.6 Cloud Agents (Dynamic Provisioning)

Jenkins can spin up agents on demand in cloud environments:

### Kubernetes Cloud

```groovy
pipeline {
    agent {
        kubernetes {
            label 'k8s-pod'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8
    command: ["cat"]
    tty: true
  - name: docker
    image: docker:20.10
    command: ["cat"]
    tty: true
  - name: jnlp
    image: jenkins/inbound-agent:latest
"""
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp .'
                }
            }
        }
    }
}
```

---

## 2.7 Scaling Jenkins

### Vertical Scaling
- Add more RAM, CPU, and disk to the master
- Master handles ~50-100 concurrent connections before needing agents

### Horizontal Scaling
- Add more permanent agents for steady workload
- Use cloud agents for burst capacity
- Distribute agents across availability zones for resilience

### Best Practices

| Practice | Reason |
|----------|--------|
| Master runs NO builds | Prevents resource contention |
| Use labels for routing | Ensures builds run on correct environment |
| Keep agents stateless | Makes agents disposable and replaceable |
| Monitor agent connectivity | Prevent silent failures |
| Pin plugin versions | Avoid compatibility issues |

---

## Key Takeaways

- Jenkins uses a master/agent architecture — the master schedules, agents execute
- Agents connect via SSH or JNLP and can run different operating systems
- Cloud agents (Kubernetes, AWS EC2, Azure) can be provisioned dynamically
- Labels route pipeline stages to the correct agent types
- For production, the master should only coordinate — never run builds

---

## Next Steps

→ Continue to [Chapter 3: Configuration & Management]({{< relref "03-jenkins-configuration-and-management" >}})
