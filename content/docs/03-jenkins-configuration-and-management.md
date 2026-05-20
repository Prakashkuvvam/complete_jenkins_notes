---
title: "Ch 3: Configuration & Management"
weight: 3
---

# Jenkins Configuration & Management

## Learning Objectives

After reading this chapter, you will be able to:
- Manage Jenkins plugins effectively
- Configure system settings for security and performance
- Set up credentials for secure access to external tools
- Implement role-based access control (RBAC)
- Create and manage Jenkins folders
- Back up and restore Jenkins configuration

---

## 3.1 Plugin Management

Plugins are the heart of Jenkins' extensibility. With 1,800+ plugins available, you can integrate Jenkins with virtually any tool in the DevOps ecosystem.

### Essential Plugins

| Plugin | Purpose |
|--------|---------|
| **Pipeline** | Pipeline as code (Jenkinsfile) support |
| **Git** | Git integration for SCM |
| **Blue Ocean** | Modern UI for pipelines |
| **Docker Pipeline** | Docker build/push within pipelines |
| **Kubernetes** | Dynamic agent provisioning on K8s |
| **Credentials Binding** | Secure credential injection |
| **Role-based Strategy** | Fine-grained access control |
| **Configuration as Code** | YAML-based Jenkins configuration |
| **Job DSL** | Programmatic job creation |
| **Slack/Email** | Build notifications |

### Installing Plugins

**Via UI:** Dashboard → Manage Jenkins → Plugins → Available

**Via CLI:**
```bash
# Install plugin via Jenkins CLI
java -jar jenkins-cli.jar -s http://localhost:8080 \
  install-plugin git pipeline-stage-view \
  -deploy
```

**Via JCasC (YAML):**
```yaml
plugins:
  - artifactId: "git"
  - artifactId: "pipeline-stage-view"
  - artifactId: "docker-workflow"
```

> **Best Practice:** Pin plugin versions and test upgrades in a staging environment before applying to production Jenkins.

---

## 3.2 System Configuration

### Configure Jenkins URL

Dashboard → Manage Jenkins → System → Jenkins URL

```yaml
# Set URL for proper link generation
http://jenkins.example.com:8080/
```

### Environment Variables

Set global environment variables available to all pipelines:

Dashboard → Manage Jenkins → System → Global Properties

```
KEY=JAVA_HOME
VALUE=/usr/lib/jvm/java-17-openjdk-amd64
```

### Tools Configuration

Configure paths to build tools:

Dashboard → Manage Jenkins → Tools

```yaml
# JDK Installations
- Name: "JDK 17"
  Install automatically: true
  Version: "jdk-17.0.6+10"

# Maven Installations
- Name: "Maven 3.9"
  Install automatically: true
  Version: "3.9.1"

# Go Installations
- Name: "Go 1.21"
  Install automatically: true
  Version: "1.21.0"
```

---

## 3.3 Credential Management

Credentials allow Jenkins to authenticate with external services securely.

### Credential Types

| Type | Usage |
|------|-------|
| **Username with password** | Git, web services, databases |
| **SSH key** | Git repositories, remote servers |
| **Secret text** | API tokens, secret keys |
| **Certificate** | TLS/SSL certificates |
| **File** | Uploaded credential files |

### Creating Credentials

Dashboard → Manage Jenkins → Credentials → System → Global credentials → Add Credentials

### Using Credentials in Pipelines

```groovy
pipeline {
    agent any
    environment {
        // Bind credentials to environment variables
        DOCKER_CREDS = credentials('docker-hub-creds')
        GIT_TOKEN = credentials('github-token')
    }
    stages {
        stage('Checkout') {
            steps {
                // Using SSH key
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'git@github.com:org/repo.git',
                        credentialsId: 'github-ssh-key'
                    ]]
                ])
            }
        }
        stage('Docker Login') {
            steps {
                sh "echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin"
            }
        }
    }
}
```

### Using withCredentials for Sensitive Values

```groovy
stage('Deploy') {
    steps {
        withCredentials([string(
            credentialsId: 'prod-api-key',
            variable: 'API_KEY'
        )]) {
            sh 'deploy.sh --api-key $API_KEY'
        }
    }
}
```

---

## 3.4 Security & Access Control

### Authentication

Jenkins supports several authentication realms:

| Realm | Description |
|-------|-------------|
| **Jenkins' own user database** | Built-in, simple setup |
| **LDAP/Active Directory** | Enterprise SSO |
| **GitHub/GitLab OAuth** | Social login |
| **SAML/SSO** | Enterprise identity providers |

### Authorization

**Role-Based Strategy** is the recommended authorization model:

1. Install **Role-based Authorization Strategy** plugin
2. Go to Dashboard → Manage Jenkins → Configure Global Security
3. Set Authorization to "Role-Based Strategy"
4. Configure roles:

```
Global Roles:
  - admin:     Full access
  - developer: Build jobs, read access
  - viewer:    Read-only

Project Roles:
  - team-alpha-dev:   Build, configure team-alpha/* jobs
  - team-beta-dev:    Build, configure team-beta/* jobs
  - team-alpha-view:  Read team-alpha/* jobs
```

### Matrix-Based Security

For simpler setups, use matrix-based permissions:

```groovy
// Each cell grants or denies a permission
hudson.model.Hudson.Read
hudson.model.Item.Build
hudson.model.Item.Configure
hudson.model.Item.Delete
hudson.model.Run.Update
```

---

## 3.5 Folders, Views & Organization

### Folders

Folders organize jobs hierarchically:

```
Jenkins
├── Team-Alpha
│   ├── frontend-app
│   │   ├── build
│   │   ├── test
│   │   └── deploy
│   └── backend-api
│       ├── build
│       ├── test
│       └── deploy
├── Team-Beta
│   └── mobile-app
│       ├── build-android
│       └── build-ios
└── Shared
    ├── security-scan
    └── deploy-tools
```

### Creating a Folder

```groovy
// Using Job DSL
folder('Team-Alpha') {
    displayName('Team Alpha')
    description('CI/CD pipelines for Team Alpha')
}
```

### Views

Views provide filtered perspectives of jobs:

```groovy
// List View — simple list of jobs
listView('Production Jobs') {
    jobs {
        regex('prod-.*')
    }
    columns {
        status()
        weather()
        name()
        lastSuccess()
        lastFailure()
    }
}
```

---

## 3.6 Backup & Restore

### What to Backup

| Component | Location | Notes |
|-----------|----------|-------|
| Jenkins home | `/var/jenkins_home` | Config, jobs, builds |
| Jobs config | `JENKINS_HOME/jobs/*/config.xml` | Individual job configs |
| Plugin configs | `JENKINS_HOME/plugins/*.jpi` | Installed plugins |
| Secrets | `JENKINS_HOME/secrets/` | Credentials (encrypted) |
| Users | `JENKINS_HOME/users/` | User accounts |

### Backup Script

```bash
#!/bin/bash
BACKUP_DIR="/backup/jenkins/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# Backup Jenkins home (excluding large build artifacts)
rsync -av --exclude='jobs/*/builds/*' \
  --exclude='workspace/*' \
  --exclude='logs/*' \
  /var/jenkins_home/ $BACKUP_DIR/

# Archive
tar czf "${BACKUP_DIR}.tar.gz" $BACKUP_DIR

# Optional: Upload to S3
aws s3 cp "${BACKUP_DIR}.tar.gz" s3://my-jenkins-backups/
```

### Using ThinBackup Plugin

Install the **ThinBackup** plugin to automate backups:

1. Go to Manage Jenkins → ThinBackup → Settings
2. Configure:

```
Backup directory: /var/jenkins_backups
Full backup every: 7 days
Diff backup every: 1 day
Max number of backup sets: 10
Files to exclude: */builds/*, */workspace/*
```

---

## 3.7 Monitoring Jenkins

### System Information

Dashboard → Manage Jenkins → System Information

Check:
- **Memory usage** — Heap and non-heap memory
- **Executor counts** — Busy/idle executors
- **Queue length** — Pending build requests
- **Agent status** — Online/offline agents

### Script Console

Dashboard → Manage Jenkins → Script Console

Run Groovy scripts to inspect or modify Jenkins state:

```groovy
// Check queue length
println "Queue length: ${Jenkins.instance.queue.items.size()}"

// List all executors
Jenkins.instance.computers.each { c ->
    println "Node: ${c.name} - Online: ${c.online} - Busy: ${c.countBusy}"
}

// Get memory stats
def runtime = Runtime.getRuntime()
println "Max memory: ${runtime.maxMemory() / 1024 / 1024} MB"
println "Used memory: ${(runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024} MB"
```

---

## Key Takeaways

- Plugins make Jenkins extensible — manage them carefully with version pinning
- Credentials provide secure authentication to external services
- Role-Based Strategy is the recommended authorization model for teams
- Folders and views organize jobs at scale
- Regular backups of `JENKINS_HOME` are essential for disaster recovery
- The Script Console is a powerful tool for debugging and maintenance

---

## Next Steps

→ Continue to [Chapter 4: Freestyle Projects & Build Jobs]({{< relref "04-freestyle-projects-and-build-jobs" >}})
