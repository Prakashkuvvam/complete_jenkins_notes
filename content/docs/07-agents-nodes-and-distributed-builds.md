---
title: "Ch 7: Agents, Nodes & Distributed Builds"
weight: 7
---

# Agents, Nodes & Distributed Builds

## Learning Objectives

After reading this chapter, you will be able to:
- Configure different types of Jenkins agents
- Use node labels for build routing
- Set up Docker-based build agents
- Configure Kubernetes cloud agents
- Implement ephemeral cloud agents with AWS/Azure
- Troubleshoot agent connectivity issues

---

## 7.1 Agent Types

Jenkins supports several agent connection types:

| Type | Launch Method | Use Case |
|------|---------------|----------|
| **Permanent agent** | SSH | Dedicated build servers |
| **JNLP agent** | Java Web Start | Machines behind firewalls, Windows agents |
| **Docker agent** | Docker container | Ephemeral, clean-slate builds |
| **Kubernetes pod** | Kubernetes plugin | Cloud-native, dynamic provisioning |
| **Cloud agent** | EC2/Azure plugin | Auto-scaling in cloud environments |

---

## 7.2 Agent Configuration

### SSH Agent (Linux/Unix)

1. Install **SSH Build Agents plugin**
2. Dashboard → Manage Jenkins → Nodes → New Node
3. Configure:

```
Name: linux-builder-prod
Remote root directory: /home/jenkins/agent
Labels: linux amd64 prod
Launch method: Launch agents via SSH
Host: 10.0.1.50
Port: 22
Credentials: (SSH private key)
Host Key Verification Strategy: Non verifying
Availability: Keep this agent online as much as possible
Number of executors: 4
```

### JNLP Agent (Windows/Docker)

```bash
# Method 1: Direct JNLP connection
curl -O http://jenkins:8080/jnlpJars/agent.jar
java -jar agent.jar \
  -url http://jenkins:8080 \
  -secret @secret-file \
  -name windows-builder-01 \
  -workDir "C:\jenkins\agent"

# Method 2: Docker inbound agent
docker run \
  -e JENKINS_URL=http://jenkins:8080 \
  -e JENKINS_SECRET=<SECRET> \
  -e JENKINS_AGENT_NAME=docker-agent-01 \
  -e JENKINS_AGENT_WORKDIR=/home/jenkins/agent \
  jenkins/inbound-agent:latest
```

---

## 7.3 Using Labels Effectively

Labels are the primary mechanism for routing builds to agents.

### Label Syntax

```groovy
// Basic label
agent { label 'linux' }

// AND condition - both must match
agent { label 'linux && docker' }

// OR condition - either can match
agent { label 'high-mem || gpu' }

// NOT condition - exclude certain nodes
agent { label '!spot-instance' }

// Complex combinations
agent { label '(linux || macos) && docker && !arm64' }
```

### Label Strategy

| Strategy | Labels | Example |
|----------|--------|---------|
| **By OS** | `linux`, `windows`, `macos` | Route builds by platform |
| **By capability** | `docker`, `gpu`, `ssd` | Match hardware requirements |
| **By environment** | `prod`, `staging`, `dev` | Separate build environments |
| **By project** | `backend`, `frontend`, `mobile` | Dedicated build capacity |
| **By security** | `isolated`, `trusted` | Compliance requirements |

### Setting Labels on Agents

```groovy
// Label can be set via agent configuration UI
// Or via Jenkins script console:
import jenkins.model.*
def node = Jenkins.instance.getNode('agent-01')
node.setLabelString('linux docker amd64 prod')
node.save()
```

---

## 7.4 Docker Build Agents

Docker agents provide clean, reproducible build environments.

### Docker Agent in Pipeline

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'
            label 'docker'   // Only run on agents with Docker
            args '-v /maven-cache:/root/.m2'
            reuseNode true
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

### Docker Pipeline Plugin

```groovy
stage('Multi-Container Build') {
    agent {
        docker {
            image 'node:18-alpine'
        }
    }
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Test') {
            stages {
                stage('Unit') {
                    agent {
                        docker { image 'node:18-alpine' }
                    }
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            args '--link my-db:db'
                        }
                    }
                    steps {
                        sh 'npm run test:integration'
                    }
                }
            }
        }
    }
}
```

### Docker Agent with Custom Dockerfile

```groovy
pipeline {
    agent {
        dockerfile {
            filename 'ci/Dockerfile.build'
            dir 'ci'
            args '-v /cache:/cache'
            label 'linux'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }
}
```

---

## 7.5 Kubernetes Agents

The Kubernetes plugin dynamically provisions pods as build agents.

### Installing the Plugin

1. Install **Kubernetes** plugin
2. Configure cloud: Dashboard → Manage Jenkins → Clouds → Add a new cloud → Kubernetes

```
Kubernetes URL: https://kubernetes.default.svc
Kubernetes Namespace: jenkins
Credentials: (Kube config or service account)
Jenkins URL: http://jenkins.jenkins.svc.cluster.local:8080
```

### Pod Template Configuration

```yaml
# Kubernetes Pod Template
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    args: ['$(JENKINS_SECRET)', '$(JENKINS_NAME)']
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: ['cat']
    tty: true
    resources:
      requests:
        memory: "2Gi"
        cpu: "1"
      limits:
        memory: "4Gi"
        cpu: "2"
  - name: docker
    image: docker:20.10-dind
    command: ['cat']
    tty: true
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
```

### Using Kubernetes Agents in Pipelines

```groovy
pipeline {
    agent {
        kubernetes {
            label 'k8s-build-pod'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: ["cat"]
    tty: true
  - name: node
    image: node:18-alpine
    command: ["cat"]
    tty: true
'''
            defaultContainer 'maven'
        }
    }
    stages {
        stage('Java Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Node Build') {
            steps {
                container('node') {
                    sh 'npm ci && npm run build'
                }
            }
        }
    }
}
```

---

## 7.6 Cloud Agents (AWS EC2)

The **Amazon EC2 plugin** provisions agents on demand.

### EC2 Plugin Configuration

Dashboard → Manage Jenkins → Configure System → Cloud → Add → Amazon EC2

```
AWS Region: us-east-1
AWS Access Key ID: AKIA...
Secret Access Key: ...
EC2 Key Pair's Private Key: (PEM file contents)

AMI ID: ami-0abcdef1234567890
Instance Type: t3.medium
Security Group: sg-12345678
Subnet ID: subnet-12345678
Labels: ec2-spot ec2-linux
Number of Executors: 1
```

### Spot Fleet / Mixed Instances

```yaml
# For cost optimization, use Spot instances:
Spot Instance Request: true
Bid Price: 0.05  # USD per hour
Instance Types:
  - t3.medium
  - t3.large
  - c5.large
```

---

## 7.7 Agent Offline/Online Scripts

Run scripts when an agent connects or disconnects:

### Online Script

```bash
#!/bin/bash
# Script runs when agent comes online
echo "Agent $(hostname) connected to Jenkins"
# Mount NFS share
mount -t nfs nfs-server:/exports/build-cache /mnt/build-cache
# Clean workspace
rm -rf /var/jenkins_home/workspace/*
```

### Offline Script

```bash
#!/bin/bash
# Script runs when agent goes offline
echo "Agent $(hostname) disconnected from Jenkins"
# Save build logs
rsync -av /var/jenkins_home/workspace/ /backup/workspace/
# Unmount NFS
umount /mnt/build-cache
```

---

## 7.8 Troubleshooting Agents

### Common Issues & Solutions

| Issue | Symptom | Solution |
|-------|---------|----------|
| **Agent not connecting** | "Agent is offline" in UI | Check firewall ports (50000, 22) |
| **SSH connection refused** | "java.net.ConnectException" | Verify agent is running and SSH is listening |
| **JNLP connection timeout** | Connection hangs | Check JNLP port and proxy settings |
| **Out of disk space** | Build fails with "No space left" | Configure build retention, clean workspaces |
| **Out of memory** | Jenkins crashes with OOM | Increase heap: `-Xmx8g` |
| **Clock skew** | "Agent time differs by X" | Configure NTP on all nodes |

### Agent Logs

```bash
# View agent logs from master
# Jenkins → Manage Jenkins → System Log → Agent → agent-01

# Or via script console
println Jenkins.instance.getNode('agent-01').toComputer().log

# Agent-side logs
tail -f /var/jenkins_home/remoting/logs/remoting.log
```

### Restarting Agents

```bash
# Via web UI
Dashboard → Agents → agent-name → Disconnect → Restart

# Via Jenkins CLI
java -jar jenkins-cli.jar -s http://jenkins:8080 \
  disconnect-agent agent-01 -message "Maintenance"

java -jar jenkins-cli.jar -s http://jenkins:8080 \
  connect-agent agent-01
```

---

## Key Takeaways

- Agents execute builds; the master should only coordinate
- Labels route builds to appropriate agents using boolean expressions
- Docker agents provide clean, reproducible environments
- Kubernetes agents dynamically provision pods per pipeline run
- Cloud agents (EC2, Azure) auto-scale for burst capacity
- Use online/offline scripts for agent setup and teardown
- Monitor agent connectivity and disk space regularly

---

## Next Steps

→ Continue to [Chapter 8: Source Code Management]({{< relref "08-source-code-management" >}})
