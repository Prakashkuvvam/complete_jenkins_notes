---
title: "Ch 15: Exam Preparation Guide"
weight: 15
---

# Exam Preparation Guide

## Learning Objectives

After reading this chapter, you will be able to:
- Understand the Jenkins certification exam domains
- Create an effective study plan
- Use cheat sheets for quick reference
- Identify key topics to focus on
- Practice with sample questions

---

## 15.1 Certification Overview

The **Jenkins Certified Engineer** exam covers these domains:

| Domain | Weight | Key Topics |
|--------|--------|------------|
| Installation & Configuration | 15% | Installation, plugins, system config |
| Pipeline as Code | 25% | Jenkinsfile, Declarative/Scripted, shared libs |
| Authentication & Authorization | 10% | Security realms, RBAC, credentials |
| Source Code Management | 10% | Git integration, webhooks, multi-branch |
| Builds & Testing | 15% | Freestyle, pipeline steps, test reports |
| Distributed Builds | 10% | Agents, labels, Docker/K8s agents |
| Monitoring & Maintenance | 10% | Backups, monitoring, performance |
| Security & Compliance | 5% | Secrets, audit, container security |

---

## 15.2 Key Concepts Cheat Sheet

### Pipeline Basics

```groovy
// Declarative Pipeline Structure
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Hello'
            }
        }
    }
    post {
        success { /* runs on success */ }
        failure { /* runs on failure */ }
        always  { /* always runs */ }
    }
}
```

### Common Pipeline Directives

| Directive | Purpose |
|-----------|---------|
| `agent any` | Run on any available agent |
| `agent { label 'linux' }` | Run on specific label |
| `agent { docker 'node:18' }` | Run inside Docker container |
| `when { branch 'main' }` | Conditional execution |
| `parallel { ... }` | Run stages in parallel |
| `input { ... }` | Manual approval gate |
| `tools { maven 'Maven 3.9' }` | Auto-install tools |
| `options { timeout(1) }` | Pipeline behavior settings |
| `environment { VAR = 'val' }` | Environment variables |
| `parameters { string(...) }` | Build parameters |

### Git Checkout Options

```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[url: 'https://github.com/org/repo.git']],
    extensions: [
        [$class: 'CloneOption', depth: 1],
        [$class: 'CleanCheckout'],
        [$class: 'RelativeTargetDirectory', relativeTargetDir: 'src']
    ]
])
```

### Common Steps Reference

| Step | Description |
|------|-------------|
| `sh 'command'` | Run shell command (Linux) |
| `bat 'command'` | Run batch command (Windows) |
| `echo 'message'` | Print message to console |
| `checkout scm` | Checkout from source control |
| `junit '*.xml'` | Publish JUnit test results |
| `archiveArtifacts '*.jar'` | Archive build artifacts |
| `stash/unstash` | Save/restore files between stages |
| `retry(3) { ... }` | Retry block on failure |
| `timeout(30) { ... }` | Time limit for block |
| `sleep(10)` | Pause for N seconds |
| `error('message')` | Fail the pipeline |
| `withCredentials([...]) { }` | Bind credentials securely |
| `docker.build('image')` | Build Docker image |
| `slackSend(...)` | Send Slack notification |

---

## 15.3 Quick Reference Tables

### Jenkinsfile Environment Variables

| Variable | Description |
|----------|-------------|
| `env.BUILD_NUMBER` | Current build number |
| `env.BUILD_ID` | Build ID (same as number) |
| `env.BUILD_URL` | URL of the current build |
| `env.JOB_NAME` | Full job name (e.g., `myapp/main`) |
| `env.BRANCH_NAME` | Branch being built |
| `env.GIT_COMMIT` | Commit SHA checked out |
| `env.CHANGE_ID` | PR number (if PR build) |
| `env.CHANGE_TARGET` | Target branch of PR |
| `env.WORKSPACE` | Absolute path to workspace |
| `env.NODE_NAME` | Agent node name |
| `env.EXECUTOR_NUMBER` | Executor number on node |
| `env.CURRENT_RESULT` | Current build result |
| `currentBuild.result` | Build result (SUCCESS/FAILURE/UNSTABLE) |

### Agent Labels

```groovy
// Label expressions
agent { label 'linux' }                    // Single label
agent { label 'linux && docker' }          // AND
agent { label 'high-mem || gpu' }          // OR
agent { label '!spot-instance' }           // NOT
agent { label '(linux || macos) && docker' } // Complex
```

### Post Conditions

| Condition | Runs When |
|-----------|-----------|
| `always` | Always |
| `success` | Build success |
| `failure` | Build failure |
| `unstable` | Unstable (test failures) |
| `changed` | Status changed from previous |
| `aborted` | Build manually aborted |
| `regression` | Fixed-then-broken |

---

## 15.4 Study Plan

### 2-Week Intensive Plan

| Day | Focus | Study Material |
|-----|-------|----------------|
| 1 | CI/CD concepts, Jenkins overview | [Ch 1]({{< relref "01-introduction-to-ci-cd-and-jenkins" >}}) |
| 2 | Architecture, installation | [Ch 2]({{< relref "02-jenkins-architecture-and-installation" >}}) |
| 3 | Configuration, plugins, security | [Ch 3]({{< relref "03-jenkins-configuration-and-management" >}}) |
| 4 | Freestyle projects | [Ch 4]({{< relref "04-freestyle-projects-and-build-jobs" >}}) |
| 5 | Pipeline fundamentals | [Ch 5]({{< relref "05-pipeline-fundamentals" >}}) |
| 6 | Pipeline syntax & steps | [Ch 6]({{< relref "06-pipeline-syntax-and-steps" >}}) |
| 7 | **Practice Test 1** | [Test 1]({{< relref "18-exam-practice-test-1" >}}) |
| 8 | Agents & distributed builds | [Ch 7]({{< relref "07-agents-nodes-and-distributed-builds" >}}) |
| 9 | SCM integration | [Ch 8]({{< relref "08-source-code-management" >}}) |
| 10 | Shared libraries | [Ch 9]({{< relref "09-shared-libraries-and-reusability" >}}) |
| 11 | Multi-branch & Docker/K8s | [Ch 10-11]({{< relref "10-multi-branch-pipelines" >}}) |
| 12 | **Practice Test 2** | [Test 2]({{< relref "19-exam-practice-test-2" >}}) |
| 13 | JCasC, Production | [Ch 13-14]({{< relref "13-jenkins-configuration-as-code" >}}) |
| 14 | **Practice Test 3 & 4** | [Test 3]({{< relref "20-exam-practice-test-3" >}}) · [Test 4]({{< relref "21-exam-practice-test-4" >}}) |

### 4-Week Balanced Plan

Follow the [Start Here]({{< relref "26-start-here" >}}) guide for a complete 4-week curriculum.

---

## 15.5 Sample Questions

### Question 1: Declarative Pipeline

Which of the following is the correct structure for a Declarative Pipeline?

```groovy
A. node {
     stage('Build') { echo 'Building' }
   }

B. pipeline {
     agent any
     stages {
       stage('Build') { steps { echo 'Building' } }
     }
   }

C. job('Build') {
     steps { echo 'Building' }
   }

D. build('Build') {
     sh 'echo Building'
   }
```

**Answer: B** — Declarative pipelines use the `pipeline {}` block with `agent`, `stages`, and `steps`.

### Question 2: Agent Labels

What does the label expression `linux && !arm64` mean?

```
A. Run on Linux or ARM64 agents
B. Run on Linux agents that are not ARM64
C. Run on agents that are not Linux or ARM64
D. Run on ARM64 agents only
```

**Answer: B** — The `&&` means AND, the `!` means NOT. So it matches agents with label `linux` that do NOT have label `arm64`.

### Question 3: Shared Libraries

Where should reusable pipeline functions be stored in a shared library?

```
A. src/com/company/
B. vars/
C. resources/
D. Jenkinsfile
```

**Answer: B** — The `vars/` directory stores global variables that become pipeline steps. The filename becomes the step name.

---

## 15.6 Exam Tips

### Before the Exam

- [ ] Go through all 17 chapters
- [ ] Complete all 7 practice tests (399 questions total)
- [ ] Set up a local Jenkins instance and practice
- [ ] Focus on Pipeline as Code (25% of exam)
- [ ] Review agent labels and label expressions
- [ ] Understand credential types and binding methods
- [ ] Know the JCasC YAML syntax

### During the Exam

- Read questions carefully — look for keywords like "NOT", "ALWAYS", "BEST"
- Eliminate obviously wrong answers first
- Use process of elimination for multiple choice
- Don't spend more than 2 minutes on a single question
- Flag hard questions and come back later

### Common Mistakes

| Mistake | Correct |
|---------|---------|
| Confusing Declarative vs Scripted syntax | Declarative uses `pipeline {}`, Scripted uses `node {}` |
| Thinking master should run builds | Master should only coordinate — use agents |
| Using `latest` tags in Docker | Always pin specific versions |
| Hard-coding credentials | Use `credentialsId` and `withCredentials` |
| Ignoring `post` conditions | `post` handles success/failure cleanup |
| Not using shared libraries | DRY up common pipeline code |
| Forgetting `serialVersionUID` | All library classes must be `Serializable` |

---

## Key Takeaways

- Pipeline as Code is the largest exam domain (25%)
- Master how agents, labels, and distributed builds work
- Practice with at least 4-5 practice tests before the exam
- Know the difference between Declarative and Scripted pipelines
- Understand credential types and secure secret management
- Review cheat sheets before the exam for quick reference

---

## Next Steps

→ [Take Practice Test 1]({{< relref "18-exam-practice-test-1" >}}) to assess your current knowledge

→ Continue to [Chapter 16: Interview Questions & Answers]({{< relref "16-interview-questions" >}})
