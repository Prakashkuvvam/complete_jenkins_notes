---
title: "Ch 13: Jenkins Configuration as Code (JCasC)"
weight: 13
---

# Jenkins Configuration as Code (JCasC)

## Learning Objectives

After reading this chapter, you will be able to:
- Understand the principles of Jenkins Configuration as Code
- Write JCasC YAML configuration files
- Manage plugins, security, credentials, and jobs via YAML
- Implement GitOps workflows for Jenkins configuration
- Validate and test JCasC configurations

---

## 13.1 What is JCasC?

**Jenkins Configuration as Code (JCasC)** is a plugin that lets you define Jenkins configuration in human-readable YAML files. Instead of clicking through UI screens, you declare the desired state:

### Before JCasC (Manual)
- Click through 20+ UI pages to configure Jenkins
- No version control for configuration
- Difficult to reproduce on another server
- Manual upgrade process

### After JCasC (Declarative)
```yaml
jenkins:
  systemMessage: "Jenkins configured by JCasC"
  numExecutors: 0
  mode: NORMAL
  scmCheckoutRetryCount: 3
```

### Benefits

| Benefit | Description |
|---------|-------------|
| **Version controlled** | Store YAML in Git, review changes via PR |
| **Reproducible** | Same config file = identical Jenkins setup |
| **Auditable** | Every config change has a commit history |
| **Fast onboarding** | New Jenkins instance = 1 file import |
| **Disaster recovery** | Rebuild Jenkins in minutes |

---

## 13.2 Installing JCasC

1. Install **Configuration as Code** plugin
2. Set the config file path:

```bash
# Environment variable
export CASC_JENKINS_CONFIG=/var/jenkins_home/casc_configs/jenkins.yaml

# Or using JAVA_OPTS
-Dcasc.jenkins.config=/var/jenkins_home/casc_configs/jenkins.yaml
```

3. Docker Compose with JCasC:

```yaml
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    volumes:
      - jenkins_home:/var/jenkins_home
      - ./casc_configs:/var/jenkins_home/casc_configs
    environment:
      CASC_JENKINS_CONFIG: /var/jenkins_home/casc_configs/jenkins.yaml
      JAVA_OPTS: -Djenkins.install.runSetupWizard=false
      JENKINS_ADMIN_ID: admin
      JENKINS_ADMIN_PASSWORD: admin
```

---

## 13.3 Core Configuration

### System Configuration

```yaml
jenkins:
  systemMessage: "Managed by JCasC - Do not edit via UI"
  numExecutors: 0  # Master doesn't run builds
  mode: NORMAL
  scmCheckoutRetryCount: 3
  
  # Security
  securityRealm:
    local:
      allowsSignup: false
      enableCaptcha: false
      users:
        - id: "admin"
          password: "${ADMIN_PASSWORD}"
        - id: "bot"
          password: "${BOT_PASSWORD}"
  
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"
            assignments:
              - "admin"
          - name: "developer"
            permissions:
              - "Overall/Read"
              - "Job/Read"
              - "Job/Build"
              - "Run/Update"
            assignments:
              - "authenticated"
          - name: "viewer"
            permissions:
              - "Overall/Read"
              - "Job/Read"
            assignments:
              - "anonymous"
  
  # Jenkins URL
  location:
    url: "http://jenkins.example.com:8080/"
    adminAddress: "admin@example.com"
```

### Plugin Management

```yaml
plugins:
  - artifactId: "git"
    source:
      version: "5.2.0"
  
  - artifactId: "pipeline-stage-view"
  
  - artifactId: "docker-workflow"
    source:
      version: "572.v950f58993883"
  
  - artifactId: "kubernetes"
    source:
      version: "4029.v5712230ccb_20"
  
  - artifactId: "configuration-as-code"
    source:
      version: "1775.v810dc9b_b_1dc"
  
  - artifactId: "workflow-aggregator"
  
  - artifactId: "role-strategy"

# Automatically install all listed plugins on startup
unclassified:
  location:
    url: "http://jenkins.example.com:8080/"
```

---

## 13.4 Credentials Configuration

```yaml
credentials:
  system:
    domainCredentials:
      - credentials:
          # Username/Password
          - usernamePassword:
              scope: GLOBAL
              id: "github-token"
              username: "my-github-username"
              password: "${GITHUB_TOKEN}"
              description: "GitHub personal access token"
          
          # Secret text
          - string:
              scope: GLOBAL
              id: "slack-webhook"
              secret: "${SLACK_WEBHOOK_URL}"
              description: "Slack notification webhook"
          
          # SSH key
          - basicSSHUserPrivateKey:
              scope: GLOBAL
              id: "github-ssh-key"
              username: "git"
              passphrase: ""
              privateKeySource:
                directEntry:
                  privateKey: "${GITHUB_SSH_PRIVATE_KEY}"
              description: "GitHub SSH deploy key"
          
          # File credential
          - file:
              scope: GLOBAL
              id: "kube-config"
              fileName: "kubeconfig"
              secretBytes: "${KUBECONFIG_BASE64}"
              description: "Kubernetes cluster configuration"
```

---

## 13.5 Agent Configuration

### Permanent Agents

```yaml
jenkins:
  nodes:
    - permanent:
        name: "linux-builder-01"
        remoteFS: "/home/jenkins/agent"
        labels: "linux amd64 prod"
        launcher:
          ssh:
            host: "10.0.1.50"
            port: 22
            credentialsId: "jenkins-ssh-key"
            sshHostKeyVerificationStrategy:
              manuallyProvidedKeyVerificationStrategy:
                sshHostKey: "SHA256:..."
```

### Kubernetes Cloud

```yaml
jenkins:
  clouds:
    - kubernetes:
        name: "k8s-cloud"
        serverUrl: "https://kubernetes.default.svc"
        namespace: "jenkins"
        skipTlsVerify: false
        jenkinsUrl: "http://jenkins.jenkins.svc.cluster.local:8080"
        credentialsId: "kube-config"
        templates:
          - name: "default-pod"
            label: "k8s-default"
            yaml: |
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                - name: jnlp
                  image: "jenkins/inbound-agent:latest"
                - name: maven
                  image: "maven:3.9-eclipse-temurin-17"
                  command:
                  - "cat"
                  tty: true
```

---

## 13.6 Job DSL with JCasC

Create seed jobs that generate other jobs via Job DSL:

```yaml
jobs:
  - script: >
      pipelineJob('seed-job') {
        definition {
          cpsScm {
            scm {
              git {
                remote {
                  url('https://github.com/org/jenkins-pipelines.git')
                  credentials('github-token')
                }
                branch('main')
              }
            }
            scriptPath('Jenkinsfile')
          }
        }
      }
```

Static job definitions:

```yaml
jobs:
  - file: /var/jenkins_home/casc_configs/jobs.yaml
```

---

## 13.7 Groovy Hooks

For configurations that JCasC doesn't support directly:

```yaml
# Groovy scripts that run after JCasC applies config
groovy:
  - defined:
      - !include /var/jenkins_home/casc_configs/hooks/init.groovy

# Or inline
  - defined: |
      import jenkins.model.*
      def instance = Jenkins.instance
      instance.setNumExecutors(0)
      instance.save()
```

---

## 13.8 Security & Secrets

### Using Environment Variables

```yaml
# Never hard-code secrets in YAML files
jenkins:
  securityRealm:
    local:
      users:
        - id: "admin"
          password: "${ADMIN_PASSWORD}"  # From env var
```

### Encrypted Values

For production, use encrypted values:

```bash
# Generate encrypted secret
java -jar jenkins-cli.jar encrypt "my-secret-value"
```

```yaml
credentials:
  system:
    domainCredentials:
      - credentials:
          - string:
              id: "api-key"
              secret: "{AQAAABAAAAAQDH2...x3nKk=}"  # Encrypted
```

---

## 13.9 GitOps Workflow

### Recommended Repository Structure

```
jenkins-config/
├── casc_configs/
│   ├── jenkins.yaml          # Main configuration
│   ├── plugins.yaml           # Plugin list
│   ├── credentials.yaml       # Credentials (templated)
│   └── jobs.yaml              # Job definitions
├── hooks/
│   └── init.groovy            # Post-config scripts
├── plugins.txt                # Alternative plugin list
├── Dockerfile                 # Custom Jenkins image
└── README.md
```

### Deployment Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Validate') {
            steps {
                sh 'java -jar jenkins-cli.jar validate-pipeline < jenkins.yaml'
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                // Apply config to Jenkins
                sh '''
                    curl -X POST http://jenkins:8080/configuration-as-code/checkConfig \\
                        --data-binary @casc_configs/jenkins.yaml
                '''
            }
        }
    }
}
```

---

## Key Takeaways

- JCasC defines Jenkins configuration in YAML instead of through the UI
- Store config files in Git for version control and disaster recovery
- Use environment variables (`${VAR}`) for secrets — never hard-code
- The `CASC_JENKINS_CONFIG` env var points to the config file location
- Validate YAML before applying to production Jenkins
- Use Groovy hooks for configuration that JCasC doesn't yet support

---

## Next Steps

→ Continue to [Chapter 14: Production-Grade Jenkins]({{< relref "14-production-grade-jenkins" >}})
