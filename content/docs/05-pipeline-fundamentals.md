---
title: "Ch 5: Pipeline Fundamentals"
weight: 5
---

# Pipeline Fundamentals

## Learning Objectives

After reading this chapter, you will be able to:
- Understand the difference between Declarative and Scripted pipelines
- Write a basic Jenkinsfile from scratch
- Use the most common pipeline directives
- Integrate Jenkinsfile with source code management
- Use the Pipeline Syntax Generator and Replay features

---

## 5.1 What is a Pipeline?

A **pipeline** is a series of automated steps that take code from commit to deployment. In Jenkins, pipelines are defined in a **Jenkinsfile** — a text file checked into your source code repository.

### Why Pipeline as Code?

| Benefit | Description |
|---------|-------------|
| **Version controlled** | The pipeline is code — review it, branch it, audit it |
| **Durable** | Survives Jenkins restarts and agent failures |
| **Reusable** | Share pipeline logic across projects with shared libraries |
| **Reviewable** | Pull request reviews for pipeline changes |
| **Testable** | Test pipeline changes before merging |
| **Self-documenting** | The pipeline file IS the documentation |

---

## 5.2 Declarative vs Scripted Pipelines

Jenkins supports two pipeline syntaxes:

### Declarative Pipeline

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

**Characteristics:**
- Stricter, more structured syntax
- Predefined sections: `pipeline`, `agent`, `stages`, `stage`, `steps`, `post`
- Easier to learn and read
- Better integration with Blue Ocean UI
- Most common for new projects

### Scripted Pipeline

```groovy
node {
    stage('Build') {
        echo 'Building...'
    }
    stage('Test') {
        echo 'Testing...'
    }
    stage('Deploy') {
        echo 'Deploying...'
    }
}
```

**Characteristics:**
- Full Groovy language freedom
- More powerful for complex logic
- No structural constraints
- Harder to review and maintain
- Better for advanced use cases

### Which One to Choose?

| Factor | Choose Declarative | Choose Scripted |
|--------|-------------------|-----------------|
| Team skill level | Beginners/Intermediate | Advanced Groovy developers |
| Pipeline complexity | Simple to moderate | Complex orchestration |
| Need for code review | Important | Important |
| Blue Ocean compatibility | Full support | Limited |
| Shared libraries | Supported | Full support |

> **Recommendation:** Start with Declarative. Use Scripted only when you need Groovy programming constructs that Declarative doesn't support.

---

## 5.3 Jenkinsfile Structure

A Declarative Jenkinsfile has this structure:

```groovy
pipeline {
    agent any                                   // Required: where to run
    
    parameters {                                 // Optional: build parameters
        string(name: 'BRANCH', defaultValue: 'main')
    }
    
    environment {                                // Optional: environment variables
        APP_NAME = 'my-app'
    }
    
    options {                                    // Optional: pipeline behavior options
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }
    
    triggers {                                   // Optional: automatic triggers
        pollSCM('H/5 * * * *')
    }
    
    tools {                                      // Optional: tool installations
        maven 'Maven 3.9'
    }
    
    stages {                                     // Required: define build stages
        stage('Build') {                         // Each stage is a logical phase
            steps {
                echo 'Building...'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
    }
    
    post {                                       // Optional: post-build actions
        always {
            echo 'This always runs'
        }
        success {
            echo 'This runs on success'
        }
        failure {
            echo 'This runs on failure'
        }
    }
}
```

---

## 5.4 The Jenkinsfile in Practice

### Basic Jenkinsfile (Java/Maven)

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9'
        jdk 'JDK 17'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
    }
    
    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar'
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

### Basic Jenkinsfile (Node.js)

```groovy
pipeline {
    agent any
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
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
            post {
                always {
                    junit 'reports/*.xml'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
    
    post {
        success {
            archiveArtifacts artifacts: 'dist/**/*'
        }
    }
}
```

### Basic Jenkinsfile (Python)

```groovy
pipeline {
    agent any
    
    stages {
        stage('Install') {
            steps {
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }
        
        stage('Lint') {
            steps {
                sh '. venv/bin/activate && flake8 src/'
            }
        }
        
        stage('Test') {
            steps {
                sh '. venv/bin/activate && pytest --junitxml=reports/junit.xml'
            }
            post {
                always {
                    junit 'reports/junit.xml'
                }
            }
        }
    }
}
```

---

## 5.5 Common Pipeline Directives

### Agent

Controls where the pipeline runs:

```groovy
// Run on any available agent
agent any

// Run on the master (not recommended for production)
node('master') { }

// Run on a specific label
agent { label 'linux && docker' }

// Run in a Docker container
agent {
    docker {
        image 'node:18-alpine'
        args '-v /tmp:/tmp'
    }
}

// No global agent — assign per stage
agent none
```

### Environment

Define variables available throughout the pipeline:

```groovy
environment {
    // Static values
    JAVA_HOME = '/usr/lib/jvm/java-17'
    
    // Dynamic values from shell
    BUILD_DATE = sh(script: 'date +%Y%m%d', returnStdout: true).trim()
    
    // Credentials
    DOCKER_HUB = credentials('docker-hub-creds')
    GITHUB_TOKEN = credentials('github-token')
}
```

### Options

Configure pipeline behavior:

```groovy
options {
    // Global timeout
    timeout(time: 1, unit: 'HOURS')
    
    // Add timestamps to console output
    timestamps()
    
    // Skip default checkout of SCM
    skipDefaultCheckout()
    
    // Retry on failure
    retry(3)
    
    // Build should skip if no changes
    skipStagesAfterUnstable()
    
    // Preserve stashes after build completion
    preserveStashes(buildCount: 5)
}
```

### Triggers

Define automatic triggers:

```groovy
triggers {
    // Poll SCM every 5 minutes
    pollSCM('H/5 * * * *')
    
    // Build periodically
    cron('H 2 * * *')
    
    // Trigger from upstream job
    upstream(
        upstreamProjects: 'build-base-image',
        threshold: hudson.model.Result.SUCCESS
    )
}
```

### Parameters

Make the pipeline parameterized:

```groovy
parameters {
    string(name: 'BRANCH', defaultValue: 'main',
        description: 'Git branch to build')
    
    choice(name: 'ENVIRONMENT',
        choices: ['dev', 'staging', 'production'],
        description: 'Deployment target')
    
    booleanParam(name: 'SKIP_TESTS',
        defaultValue: false,
        description: 'Skip automated tests')
    
    text(name: 'DEPLOY_NOTES',
        description: 'Release notes for deployment')
}
```

### Post Conditions

Run actions based on build outcome:

```groovy
post {
    always {
        // Always runs, regardless of outcome
        cleanWs()
        junit 'reports/**/*.xml'
    }
    
    success {
        // Only on success
        archiveArtifacts artifacts: 'dist/**/*'
        slackSend(color: 'good', message: "Build succeeded!")
    }
    
    failure {
        // Only on failure
        slackSend(color: 'danger', message: "Build failed!")
    }
    
    unstable {
        // Only on unstable (test failures)
        slackSend(color: 'warning', message: "Build unstable")
    }
    
    changed {
        // Only when the build status changes
        echo "Status changed from previous build"
    }
    
    aborted {
        // Only when manualy aborted
        echo "Build was aborted"
    }
    
    regression {
        // Only when last successful build was before this failure
        echo "Regression detected!"
    }
}
```

---

## 5.6 Pipeline Syntax Generator

Jenkins provides a visual tool to generate pipeline step code:

1. Open a pipeline job → **Pipeline Syntax**
2. Select a step (e.g., `checkout: Check out from version control`)
3. Fill in the parameters
4. Click **Generate Pipeline Script**
5. Copy the generated code into your Jenkinsfile

This is invaluable for learning syntax and exploring available steps.

---

## 5.7 Replay Feature

The **Replay** feature lets you modify a pipeline and re-run it without changing the Jenkinsfile in SCM:

1. Open any build → **Replay**
2. Edit the pipeline code in the editor
3. Click **Run**

Use cases:
- Debugging pipeline changes without committing
- Testing experimental configurations
- Fixing pipeline bugs in production quickly

---

## Key Takeaways

- Pipelines are defined as code in a Jenkinsfile checked into version control
- Declarative pipeline is recommended for most use cases — structured and readable
- The Jenkinsfile uses directives: `agent`, `stages`, `stage`, `steps`, `post`
- Use `post` conditions to handle build outcomes (success, failure, always)
- The Pipeline Syntax Generator helps you write correct pipeline code
- Replay lets you test pipeline changes without committing

---

## Next Steps

→ Continue to [Chapter 6: Pipeline Syntax & Steps]({{< relref "06-pipeline-syntax-and-steps" >}})
