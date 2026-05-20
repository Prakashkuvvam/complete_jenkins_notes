---
title: "Ch 10: Multi-Branch Pipelines"
weight: 10
---

# Multi-Branch Pipelines

## Learning Objectives

After reading this chapter, you will be able to:
- Create and configure multi-branch pipelines
- Understand branch indexing and discovery
- Configure pull request (PR) pipeline triggers
- Use organization folders for GitHub/GitLab organizations
- Handle branch-specific behaviors in a single Jenkinsfile
- Implement governance policies across branches

---

## 10.1 What is a Multi-Branch Pipeline?

A **multi-branch pipeline** automatically creates a pipeline job for every branch in a repository that contains a Jenkinsfile. When you add a new branch, Jenkins discovers it and creates a corresponding pipeline.

```
Repository: myapp
├── main          → jenkins job: myapp/main
├── develop       → jenkins job: myapp/develop
├── feature/auth  → jenkins job: myapp/feature/auth
└── release/1.0   → jenkins job: myapp/release%2F1.0
```

### Benefits

| Benefit | Description |
|---------|-------------|
| **Automatic** | New branches automatically get pipelines |
| **Consistent** | Same Jenkinsfile runs on every branch |
| **Context-aware** | Pipeline knows which branch it's running on |
| **Visual** | See all branch build statuses in one view |
| **PR integration** | Build pull requests automatically |

---

## 10.2 Creating a Multi-Branch Pipeline

### Step-by-Step

1. Dashboard → **New Item** → Enter name → **Multibranch Pipeline**
2. Configure:

```
Branch Sources:
  Git:
    Project Repository: https://github.com/org/myapp.git
    Credentials: github-token

    Behaviours:
      - Discover branches
      - Discover pull requests from origin
      - Discover pull requests from forks
      
Property strategy: All branches get the same properties
```

3. Build Configuration:

```
Mode: by Jenkinsfile
Script Path: Jenkinsfile
```

4. Scan triggers:

```
☑ Periodically if not otherwise run
  Interval: 1 minute
  
☑ Pull request scanning
```

---

## 10.3 Branch Properties & Strategies

### Branch Property Strategies

Control how properties are assigned to discovered branches:

```groovy
// All branches get the same properties (default)
propertyStrategy: 'AllBranchesSameProperties'

// Named branches get specific properties
propertyStrategy: 'NamedBranchesDifferentProperties'
```

### Suppress Automatic SCM Triggering

```groovy
pipeline {
    triggers {
        // Disable automatic triggers for all branches
        pollSCM('')
    }
}
```

---

## 10.4 Branch-Specific Pipeline Behavior

### Using env.BRANCH_NAME

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'deploy.sh production'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'release/*'
                }
            }
            steps {
                sh 'deploy.sh staging'
            }
        }
    }
    
    post {
        success {
            script {
                // Different notifications per branch type
                switch (env.BRANCH_NAME) {
                    case 'main':
                        slackSend(color: 'good', 
                            message: "Production deployed: ${env.JOB_NAME}")
                        break
                    case 'develop':
                        slackSend(color: 'good', 
                            message: "Staging deployed (develop)")
                        break
                    default:
                        echo "Feature branch build OK"
                }
            }
        }
    }
}
```

### Common Branch Patterns

```groovy
pipeline {
    agent any
    environment {
        // Derive environment from branch
        DEPLOY_ENV = env.BRANCH_NAME == 'main' ? 'production' :
                     env.BRANCH_NAME.startsWith('release/') ? 'staging' :
                     env.BRANCH_NAME == 'develop' ? 'dev' :
                     'ephemeral'
        
        // Set version from branch
        VERSION = env.BRANCH_NAME == 'main' ? 
                  sh(script: 'git describe --tags', returnStdout: true).trim() :
                  "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                echo "Building for ${DEPLOY_ENV} as version ${VERSION}"
                sh 'mvn clean package'
            }
        }
        
        stage('Deploy') {
            when {
                expression { DEPLOY_ENV != 'ephemeral' }
            }
            steps {
                sh "deploy.sh --env ${DEPLOY_ENV} --version ${VERSION}"
            }
        }
    }
}
```

---

## 10.5 Pull Request Pipelines

### PR Discovery Configuration

```groovy
// Discover PRs from origin (same repo)
Behaviours:
  - Discover pull requests from origin
    Strategy:
      - Merging the pull request with the current target branch version
      - The current pull request revision
      - Two merged with current target branch revision

// Discover PRs from forks (external contributors)
Behaviours:
  - Discover pull requests from forks
    Strategy:
      - Merging the pull request with the current target branch version
    Trust:
      - Everyone (high risk)
      - Permission users (safe)
      - Nobody (high security)
```

### PR Status in Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('PR Validation') {
            when {
                triggeredBy 'PullRequestBranch'
            }
            steps {
                echo "Running PR validation"
                
                // Check if PR targets a protected branch
                script {
                    if (env.CHANGE_TARGET == 'main') {
                        echo "PR targets main — running full test suite"
                    }
                }
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Require Approval') {
            when {
                branch 'PR-*'
            }
            input {
                message "Approve for merge?"
                ok "Approve"
            }
            steps {
                echo "PR approved!"
            }
        }
    }
    
    post {
        success {
            // Update GitHub commit status
            step([
                $class: 'GitHubCommitStatusSetter',
                contextSource: [
                    $class: 'ManuallyCommitContextSource',
                    context: 'ci/jenkins'
                ],
                statusResultSource: [
                    $class: 'ConditionalStatusResultSource',
                    results: [[
                        condition: 'SUCCESS',
                        message: 'Build passed',
                        state: 'SUCCESS'
                    ]]
                ]
            ])
        }
        failure {
            step([...])  // Set failure status on GitHub
        }
    }
}
```

---

## 10.6 Organization Folders

**Organization folders** extend multi-branch pipelines to an entire GitHub/GitLab organization:

### GitHub Organization Folder

1. Dashboard → **New Item** → **GitHub Organization**
2. Configure:

```
GitHub organization: my-company
Owner: my-company
Scan credentials: github-api-token

Behaviours:
  - Discover all repositories
  - Auto-generate job names
  - Filter repositories by name regex

Scan organization triggers:
  ☑ Periodically
  Interval: 1 hour
  
  ☑ Webhook notifications
```

### Resulting Structure

```
Jenkins
└── my-company (GitHub Organization)
    ├── repo-1
    │   ├── main
    │   ├── develop
    │   └── feature/abc
    ├── repo-2
    │   ├── main
    │   └── develop
    └── ...
```

---

## 10.7 Webhook Integration for Auto-Scanning

### GitHub Webhook for Organization

1. GitHub org → Settings → Webhooks → Add webhook:

```
Payload URL: http://jenkins.example.com/github-webhook/
Events: Push, Pull Request, Branch creation/deletion
```

2. Jenkins will automatically scan the organization when it receives webhooks.

### GitLab Webhook for Group

```
URL: http://jenkins.example.com/project/ORG_NAME
Secret Token: <token>
Triggers: Push events, Merge request events
```

---

## 10.8 Indexing & Performance

### Indexing Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Periodic scanning** | Scan every N minutes | Simple setups |
| **Webhook-triggered** | Scan on push/PR events | High-frequency repos |
| **Manual only** | Scan on demand | Stable repositories |

### Reducing Indexing Overhead

```groovy
// Filter branches to reduce indexing workload
Behaviours:
  - Filter by name (with wildcards)
    Include: 'main develop release/* feature/*'
    Exclude: 'renovate/** wip/**'
    
  - Prune old branches
    Days to keep: 30
    Number to keep: 20
```

### Branch Indexing Best Practices

1. **Use webhooks** instead of frequent polling
2. **Exclude WIP/Renovate branches** from discovery
3. **Prune stale branches** automatically
4. **Limit scan depth** for repositories with many branches
5. **Set appropriate scan intervals** (1-5 minutes for active repos, hourly for stable ones)

---

## Key Takeaways

- Multi-branch pipelines auto-create pipeline jobs for every branch with a Jenkinsfile
- Use `env.BRANCH_NAME` and `when { branch ... }` for branch-specific behavior
- PR pipelines validate pull requests before merging
- Organization folders extend multi-branch to entire GitHub/GitLab orgs
- Webhooks provide instant index triggers; periodic scanning is a fallback
- Filter branches to reduce indexing overhead in large repositories

---

## Next Steps

→ Continue to [Chapter 11: Docker & Kubernetes Integration]({{< relref "11-jenkins-docker-and-kubernetes" >}})
