---
title: "Ch 4: Freestyle Projects & Build Jobs"
weight: 4
---

# Freestyle Projects & Build Jobs

## Learning Objectives

After reading this chapter, you will be able to:
- Create and configure freestyle projects in Jenkins
- Set up various build triggers (SCM polling, cron, webhooks)
- Implement build steps with shell scripts and plugins
- Configure post-build actions and notifications
- Archive artifacts and publish test reports
- Understand when to use freestyle vs pipeline projects

---

## 4.1 What is a Freestyle Project?

A **freestyle project** is Jenkins' simplest job type. It provides a graphical UI for configuring build steps without writing code. While pipelines are now the recommended approach, freestyle projects are still useful for:

- **Simple build jobs** that don't need complex logic
- **Users unfamiliar with Groovy** or pipeline syntax
- **Legacy environments** that predate pipeline adoption
- **Quick prototypes** before converting to pipelines

### Freestyle vs Pipeline

| Feature | Freestyle | Pipeline |
|---------|-----------|----------|
| **Configuration** | GUI forms | Code (Jenkinsfile) |
| **Version control** | No | Yes |
| **Complex logic** | Limited | Full programming |
| **Reusability** | Low | High (shared libraries) |
| **Debugging** | Easy for simple cases | Rich replay capability |
| **Best for** | Simple, one-off jobs | Complex, multi-stage pipelines |

---

## 4.2 Creating a Freestyle Project

### Step 1: New Item

Dashboard → New Item → Enter name → **Freestyle project**

### Step 2: General Configuration

```
Project name: build-backend-service
Description: Builds the backend Java service

☑ Discard old builds
  Strategy: Log rotation
  Max # of builds to keep: 10
  Max # of days to keep builds: 30

☑ This project is parameterized
  - String parameter: BRANCH (default: main)
  - Choice parameter: ENV (dev, staging, production)
  - Boolean parameter: SKIP_TESTS

☑ Restrict where this project can be run
  Label Expression: linux && docker
```

### Step 3: Source Code Management

```
Source Code Management: Git
  Repository URL: https://github.com/myorg/backend-service.git
  Credentials: github-token
  Branches to build: */${BRANCH}

  ☑ Clean checkout (sparse checkout)
  ☑ Git LFS (Large File Storage) pull

  Additional Behaviours:
    - Check out to specific local branch: main
    - Prune stale remote-tracking branches
    - Advanced clone behaviours
      - Depth of clone: 1 (shallow clone)
      - Timeout: 10 minutes
```

### Step 4: Build Triggers

```
Build Triggers:
  ☐ Build periodically
    Schedule: H 2 * * * (daily at ~2 AM)
    
  ☐ Poll SCM
    Schedule: H/5 * * * * (every 5 minutes)
    
  ☐ GitHub hook trigger for GITScm polling
  ☐ Trigger builds remotely (e.g., from scripts)
    Authentication Token: MY_SECRET_TOKEN
```

### Step 5: Build Environment

```
Build Environment:
  ☑ Delete workspace before build starts
  ☑ Add timestamps to the Console Output
  ☑ Use secret text(s) or file(s)
    - Bindings: DEPLOY_KEY (secret text)
  ☐ Set Build Name
    Build Name: #${BUILD_NUMBER}-${BRANCH}
```

### Step 6: Build Steps

```
Build:
  Step 1: Execute shell
    Command:
      #!/bin/bash
      set -e
      echo "Building branch: ${BRANCH}"
      mvn clean compile -q
      
  Step 2: Execute shell
    Command:
      #!/bin/bash
      if [ "$SKIP_TESTS" != "true" ]; then
        mvn test -q
      fi
      
  Step 3: Execute shell
    Command:
      #!/bin/bash
      mvn package -DskipTests=true
      echo "Build complete: target/myapp.jar"
```

---

## 4.3 Build Triggers in Detail

### Poll SCM

Checks the repository for changes on a schedule:

```cron
# Every 5 minutes during business hours
H/5 9-17 * * 1-5

# Every hour on weekends
H * * * 6,0

# Every 15 minutes for critical projects
H/15 * * * *
```

The `H` (hash) distributes load — Jenkins hashes the job name to pick a minute offset.

### GitHub Webhooks

1. In GitHub repo: **Settings → Webhooks → Add webhook**
2. Payload URL: `http://jenkins.example.com/github-webhook/`
3. Content type: `application/json`
4. Events: **Just the push event** (or select individual events)

### Triggering Remotely

```bash
# Trigger build via curl
curl -X POST \
  http://jenkins.example.com/job/build-backend-service/build \
  --user admin:API_TOKEN \
  --data-urlencode "json={\"parameter\": [{\"name\":\"BRANCH\", \"value\":\"feature/new-auth\"}]}"
```

---

## 4.4 Post-Build Actions

Post-build actions run after the build completes, regardless of success or failure:

### Archiving Artifacts

```
Post-build Actions:
  ☑ Archive the artifacts
    Files to archive: target/*.jar, Dockerfile
    ☐ Archive only if build is successful
```

### Publishing Test Reports

```
Post-build Actions:
  ☑ Publish JUnit test result report
    Test report XMLs: target/surefire-reports/*.xml
    ☐ Retain long standard output/error
    ☑ Health report amplification
```

### Notifications

```
Post-build Actions:
  ☑ E-mail Notification
    Recipients: dev-team@example.com
    ☑ Send e-mail for every unstable build
    ☑ Send separate e-mails to individuals who broke the build
    
  ☑ Slack Notifications
    ☑ Notify Success
    ☑ Notify Failure
    ☑ Notify Unstable
    Channel: #jenkins-builds
```

### Triggering Downstream Jobs

```
Post-build Actions:
  ☑ Build other projects
    Projects to build: deploy-to-staging
    ☑ Trigger only if build is stable
    ☐ Trigger even if the build is unstable
    ☐ Trigger even if the build fails
```

---

## 4.5 Parameterized Builds

Parameters make builds dynamic and reusable:

### String Parameter

```
Name: BRANCH
Default Value: main
Description: Git branch to build
```

### Choice Parameter

```
Name: ENVIRONMENT
Choices:
  dev
  staging
  production
Description: Deployment target environment
```

### File Parameter

Upload a file during build triggering:

```
Name: CONFIG_FILE
Description: Custom configuration file to use during build
```

### Active Choices (with Active Choices Plugin)

```groovy
// Groovy script that dynamically populates choices
def branches = ['git ls-remote -h https://github.com/myorg/repo.git']()
branches.execute().text.readLines().collect {
    it.split('\t')[1].replace('refs/heads/', '')
}.sort()
```

---

## 4.6 Build Wrappers

Build wrappers add capabilities around the build:

| Wrapper | Purpose |
|---------|---------|
| **Abort if stuck** | Auto-abort builds exceeding time limit |
| **Inject environment variables** | Set variables from files or properties |
| **Timestamper** | Add timestamps to console output |
| **Color ANSI Console Output** | Colorize terminal output |
| **Xvfb (Virtual Display)** | Run GUI tests requiring a display |
| **With Ant** | Set up Ant build tool |
| **Prepare dependencies** | Pre-install dependencies |

```bash
# Example: Timestamper output
10:15:30 Cloning repository https://github.com/myorg/repo.git
10:15:32 Fetching upstream changes from origin
10:15:45 Starting build...
10:17:02 Build successful
```

---

## 4.7 Viewing Build Results

### Console Output

Full log of everything that happened during the build:

```
Started by user admin
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/build-backend-service
Cloning repository https://github.com/myorg/repo.git
> git rev-parse --resolve-git-dir /var/jenkins_home/workspace/.../.
.git #
Checkout: .../workspace/build-backend-service
...
Finished: SUCCESS
```

### Build Status

| Status | Color | Meaning |
|--------|-------|---------|
| **SUCCESS** | Blue/Green | Build passed all steps |
| **UNSTABLE** | Yellow | Build completed but tests failed |
| **FAILURE** | Red | Build steps failed |
| **ABORTED** | Gray | Build was manually stopped |
| **NOT BUILT** | White/Empty | No build has run yet |

### Test Results

Dashboard → Job → Specific Build → Test Result

Shows:
- Total tests run / passed / failed / skipped
- Test duration
- Stack traces for failures
- Test history trends

---

## 4.8 Converting Freestyle to Pipeline

When your freestyle project outgrows the GUI, convert it:

```groovy
pipeline {
    agent { label 'linux && docker' }
    
    parameters {
        string(name: 'BRANCH', defaultValue: 'main')
        choice(name: 'ENV', choices: ['dev', 'staging', 'production'])
        booleanParam(name: 'SKIP_TESTS', defaultValue: false)
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH}"]],
                    userRemoteConfigs: [[url: 'https://github.com/myorg/repo.git',
                        credentialsId: 'github-token']]
                ])
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile -q'
            }
        }
        
        stage('Test') {
            when { expression { !params.SKIP_TESTS } }
            steps {
                sh 'mvn test -q'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
    }
    
    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar'
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
```

---

## Key Takeaways

- Freestyle projects use GUI configuration — best for simple, one-off jobs
- Build triggers can run on schedule, SCM changes, or remote API calls
- Post-build actions handle artifacts, notifications, and downstream jobs
- Parameters make freestyle jobs reusable across branches and environments
- Convert to pipelines when you need version control, complex logic, or reusability

---

## Next Steps

→ Continue to [Chapter 5: Pipeline Fundamentals]({{< relref "05-pipeline-fundamentals" >}})
