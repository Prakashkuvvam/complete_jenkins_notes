---
title: "Ch 8: Source Code Management"
weight: 8
---

# Source Code Management

## Learning Objectives

After reading this chapter, you will be able to:
- Configure Git SCM integration in Jenkins
- Set up webhook triggers with GitHub, GitLab, and Bitbucket
- Use credential binding for secure SCM access
- Work with multiple repositories in a single pipeline
- Implement Gerrit Code Review integration
- Configure SCM polling strategies

---

## 8.1 Git Integration

### Basic Git Checkout

```groovy
stage('Checkout') {
    steps {
        // Simplest checkout (uses credentials from job config)
        checkout scm
        
        // Explicit checkout with configuration
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/main']],
            userRemoteConfigs: [[
                url: 'https://github.com/myorg/myapp.git',
                credentialsId: 'github-token'
            ]],
            extensions: [
                [$class: 'RelativeTargetDirectory', relativeTargetDir: 'myapp'],
                [$class: 'CloneOption', depth: 1, shallow: true],
                [$class: 'CleanCheckout'],
                [$class: 'PruneStaleBranch']
            ]
        ])
    }
}
```

### Git Configuration Options

```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '**/feature/*']],  // Match multiple branches
    userRemoteConfigs: [[
        url: 'https://github.com/org/repo.git',
        credentialsId: 'github-creds',
        name: 'origin',
        refspec: '+refs/heads/*:refs/remotes/origin/*'
    ]],
    extensions: [
        [$class: 'CloneOption', depth: 1],       // Shallow clone
        [$class: 'CleanBeforeCheckout'],           // Clean workspace
        [$class: 'SubmoduleOption',
            recursiveSubmodules: true,
            trackingSubmodules: true],
        [$class: 'GitLFSPull'],                    // LFS support
        [$class: 'SparseCheckoutPaths',            // Checkout specific paths
            sparseCheckoutPaths: [[path: 'src/']]],
        [$class: 'CheckoutOption', timeout: 30],   // Timeout in minutes
        [$class: 'LocalBranch', localBranch: '**'] // Create local branch
    ]
])
```

---

## 8.2 Multiple Repositories

### Repo 1: Application + Repo 2: Infrastructure

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout All Repos') {
            steps {
                // Main application repo
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/org/myapp.git',
                        credentialsId: 'github-token']],
                    extensions: [[$class: 'RelativeTargetDirectory',
                        relativeTargetDir: 'myapp']]
                ])
                
                // Infrastructure repo
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/org/infra.git',
                        credentialsId: 'github-token']],
                    extensions: [[$class: 'RelativeTargetDirectory',
                        relativeTargetDir: 'infra']]
                ])
            }
        }
        
        stage('Build') {
            steps {
                dir('myapp') {
                    sh 'make build'
                }
            }
        }
        
        stage('Deploy Infrastructure') {
            steps {
                dir('infra') {
                    sh 'terraform apply'
                }
            }
        }
    }
}
```

### Using Git Commands Directly

```groovy
stage('Custom Git Operations') {
    steps {
        sh '''
            # Clone with specific options
            git clone --depth 1 --branch main \
                https://github.com/org/myapp.git
            
            cd myapp
            
            # Get commit info
            echo "Commit: $(git rev-parse HEAD)"
            echo "Branch: $(git rev-parse --abbrev-ref HEAD)"
            echo "Author: $(git log -1 --pretty=format:'%an')"
            echo "Message: $(git log -1 --pretty=format:'%s')"
            
            # Get changed files
            git diff --name-only HEAD~1 HEAD
        '''
    }
}
```

---

## 8.3 GitHub Webhook Integration

### Setting Up GitHub Webhook

1. **GitHub side** — Repo → Settings → Webhooks → Add webhook:

```
Payload URL: http://jenkins.example.com/github-webhook/
Content type: application/json
Secret: (optional shared secret)
Events: Push, Pull Request
```

2. **Jenkins side** — Install **GitHub Integration** plugin

3. **Job configuration:**
```
GitHub project: https://github.com/org/repo
Build Triggers: ☑ GitHub hook trigger for GITScm polling
```

### Webhook Events & Actions

```groovy
pipeline {
    agent any
    triggers {
        // Trigger on push
        githubPush()
        
        // Or for specific event types
        issueCommentTrigger('.*test this please.*')
        
        // Pull request triggers
        pullRequestEvent()
    }
}
```

---

## 8.4 GitLab Integration

### GitLab Webhook

1. Install **GitLab Plugin**
2. GitLab project → Settings → Webhooks:

```
URL: http://jenkins.example.com/project/PROJECT_NAME
Secret Token: <shared secret>
Trigger: Push events, Merge Request events
```

### GitLab Configuration in Jenkins

Dashboard → Manage Jenkins → Configure System → GitLab

```
Connection name: GitLab-Corp
GitLab host URL: https://gitlab.company.com
Credentials: gitlab-api-token
```

### GitLab Pipeline Triggers

```groovy
pipeline {
    agent any
    triggers {
        // GitLab webhook triggers
        gitlab(triggerOnPush: true,
            triggerOnMergeRequest: true,
            triggerOpenMergeRequestOnPush: 'never',
            acceptMergeRequestOnSuccess: false,
            branchFilterType: 'RegexBasedFilter',
            branchFilterRegex: '^(main|release/.*)$')
    }
    stages {
        stage('Build') {
            steps {
                echo "Build triggered by GitLab ${env.gitlabSourceBranch}"
            }
        }
    }
}
```

---

## 8.5 Credentials for SCM

### SSH Key Authentication

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout with SSH') {
            steps {
                // Use SSH key credential
                sshagent(['github-ssh-key']) {
                    sh 'git clone git@github.com:org/private-repo.git'
                }
            }
        }
    }
}
```

### Token-Based Authentication

```groovy
pipeline {
    agent any
    environment {
        // Bind GitHub token
        GIT_TOKEN = credentials('github-token')
    }
    stages {
        stage('Checkout') {
            steps {
                sh """
                    git clone https://x-access-token:${GIT_TOKEN}@github.com/org/repo.git
                    cd repo
                    git checkout ${BRANCH_NAME}
                """
            }
        }
    }
}
```

### Credential Patterns for Different Providers

```groovy
// GitHub App Installation token
withCredentials([string(
    credentialsId: 'github-app-token',
    variable: 'GITHUB_TOKEN'
)]) {
    sh 'git clone https://x-access-token:$GITHUB_TOKEN@github.com/org/repo.git'
}

// BitBucket App Password
withCredentials([usernamePassword(
    credentialsId: 'bitbucket-creds',
    usernameVariable: 'BB_USER',
    passwordVariable: 'BB_PASS'
)]) {
    sh 'git clone https://$BB_USER:$BB_PASS@bitbucket.org/org/repo.git'
}

// Azure DevOps PAT
withCredentials([string(
    credentialsId: 'azure-devops-pat',
    variable: 'AZURE_PAT'
)]) {
    sh 'git clone https://pat:$AZURE_PAT@dev.azure.com/org/project/_git/repo'
}
```

---

## 8.6 SCM Polling Strategies

### Polling Options

```groovy
pipeline {
    agent any
    triggers {
        // Standard polling
        pollSCM('H/5 * * * *')
        
        // Less frequent for stable branches
        pollSCM('H */2 * * *')
        
        // No polling — rely on webhooks only
        // (omit pollSCM entirely)
    }
}
```

### Hybrid Approach

```groovy
pipeline {
    agent any
    triggers {
        // Webhook triggers for immediate builds
        githubPush()
        
        // Fallback polling every hour
        pollSCM('H * * * *')
    }
}
```

---

## 8.7 Change Detection

### Detecting Changed Files

```groovy
pipeline {
    agent any
    stages {
        stage('Conditional Build') {
            when {
                changeset "src/**"
            }
            steps {
                echo "Source files changed — building"
                sh 'make build'
            }
        }
        
        stage('Infrastructure Changes') {
            when {
                changeset "terraform/**"
            }
            steps {
                echo "Infrastructure changed — applying"
                sh 'terraform apply -auto-approve'
            }
        }
        
        stage('Documentation Only') {
            when {
                changeset "docs/**"
            }
            steps {
                echo "Only documentation changed — skipping build"
            }
        }
    }
}
```

### Using Changelog

```groovy
stage('Process Changes') {
    steps {
        script {
            // Get the list of changed files
            def changedFiles = checkout(scm).getChanges()
            
            changedFiles.each { change ->
                echo "Changed: ${change.path} (${change.editType.name})"
                
                if (change.path.endsWith('.groovy')) {
                    echo "Pipeline code changed!"
                }
            }
            
            // Check if specific files changed
            def hasDockerChanges = changedFiles.any {
                it.path.startsWith('docker/')
            }
            
            if (hasDockerChanges) {
                echo "Docker files changed — triggering Docker build"
            }
        }
    }
}
```

---

## 8.8 Gerrit Code Review Integration

### Gerrit Trigger

```groovy
pipeline {
    triggers {
        gerrit(
            triggerOn: [
                patchsetCreated: true,
                commentAdded: true,
                changeMerged: false
            ],
            projects: [[
                compareType: 'PLAIN',
                pattern: 'myapp',
                branches: [[
                    compareType: 'PLAIN',
                    pattern: '**'
                ]]
            ]]
        )
    }
    stages {
        stage('Verify') {
            steps {
                sh 'mvn verify'
            }
        }
    }
    post {
        success {
            gerritReview(
                verified: 1,
                message: 'Build passed',
                score: 1
            )
        }
        failure {
            gerritReview(
                verified: -1,
                message: 'Build failed',
                score: -1
            )
        }
    }
}
```

---

## Key Takeaways

- Use `checkout scm` for simple checkouts; explicit checkout for more control
- Multiple repos can be checked out with `RelativeTargetDirectory` extensions
- Webhooks provide instant build triggers — prefer over SCM polling
- Use SSH keys or tokens for secure SCM authentication
- `changeset` condition controls stage execution based on changed file patterns
- Gerrit integration provides code review verification results

---

## Next Steps

→ Continue to [Chapter 9: Shared Libraries & Reusability]({{< relref "09-shared-libraries-and-reusability" >}})
