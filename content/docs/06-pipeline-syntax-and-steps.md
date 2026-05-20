---
title: "Ch 6: Pipeline Syntax & Steps"
weight: 6
---

# Pipeline Syntax & Steps

## Learning Objectives

After reading this chapter, you will be able to:
- Write advanced pipeline stages with parallel execution
- Use all major built-in pipeline steps effectively
- Implement conditional execution with `when` directives
- Handle inputs, timeouts, and retries in pipelines
- Use matrix and parallel execution strategies

---

## 6.1 Stages & Steps

### Stage vs Step

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {                      // A stage is a logical phase
            steps {
                echo 'Compiling...'            // A step is a single action
                sh 'mvn compile'
                echo 'Done compiling'
            }
        }
    }
}
```

**Stages** organize the pipeline into logical phases. **Steps** are the actual actions within each stage.

### Step 0: Built-in Steps

| Category | Common Steps |
|----------|-------------|
| **Shell** | `sh`, `bat`, `powershell` |
| **SCM** | `checkout`, `git` |
| **Utilities** | `echo`, `sleep`, `writeFile`, `readFile`, `fileExists` |
| **Archiving** | `archiveArtifacts`, `stash`, `unstash` |
| **Testing** | `junit`, `findbugs`, `checkstyle`, `publishHTML` |
| **Notifications** | `slackSend`, `emailext`, `step` |
| **Docker** | `docker.build`, `docker.withRegistry`, `docker.image` |
| **Tools** | `tool`, `withMaven`, `withGradle` |
| **Credentials** | `withCredentials`, `withAWS` |

---

## 6.2 Shell Steps

### sh (Linux/macOS)

```groovy
stage('Shell Commands') {
    steps {
        // Simple command
        sh 'ls -la'
        
        // Multi-line script
        sh '''
            set -e
            echo "Building version ${BUILD_NUMBER}"
            mvn clean package -q
            echo "Build complete"
        '''
        
        // Capture output
        script {
            def version = sh(
                script: 'git describe --tags',
                returnStdout: true
            ).trim()
            echo "Version: ${version}"
        }
        
        // Check exit code
        script {
            def status = sh(
                script: 'make test',
                returnStatus: true
            )
            if (status != 0) {
                error "Tests failed with exit code ${status}"
            }
        }
    }
}
```

### bat (Windows)

```groovy
stage('Windows Build') {
    agent { label 'windows' }
    steps {
        bat 'dir /B'
        bat """
            set JAVA_HOME=C:\\Program Files\\Java\\jdk-17
            mvn clean package -q
        """
    }
}
```

### powershell (Cross-Platform)

```groovy
stage('PowerShell') {
    steps {
        powershell '''
            $version = git describe --tags
            Write-Host "Building version: $version"
            # Invoke build
            & "C:\\Program Files\\dotnet\\dotnet.exe" build
        '''
    }
}
```

---

## 6.3 Conditional Execution (when)

The `when` directive controls whether a stage runs:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'make build' }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps { sh 'deploy.sh production' }
        }
        
        stage('Deploy to Staging') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'release/*'
                }
            }
            steps { sh 'deploy.sh staging' }
        }
        
        stage('Run Integration Tests') {
            when {
                allOf {
                    branch 'main'
                    triggeredBy 'TimerTrigger'
                }
            }
            steps { sh 'make integration-tests' }
        }
        
        stage('Run Expensive Tests') {
            when {
                expression {
                    params.RUN_EXPENSIVE_TESTS == true
                }
                beforeAgent true  // Evaluate before allocating agent
            }
            steps { sh 'make expensive-tests' }
        }
        
        stage('Notify on Failure') {
            when {
                not { equals expected: 'SUCCESS', actual: currentBuild.result }
            }
            steps { slackSend(color: 'danger', message: 'Build failed') }
        }
    }
}
```

### when Conditions Summary

| Condition | Description |
|-----------|-------------|
| `branch` | Match branch pattern (e.g., `'main'`, `'feature/*'`, `'*/**'`) |
| `tag` | Match tag pattern (e.g., `'v*'`) |
| `environment` | Check environment variable value |
| `expression` | Evaluate a Groovy expression |
| `not` | Negate another condition |
| `allOf` | All conditions must be true (AND) |
| `anyOf` | Any condition must be true (OR) |
| `changeSet` | Match file change patterns |
| `triggeredBy` | Check what triggered the build |
| `beforeAgent` | Evaluate before allocating agent node |
| `beforeInput` | Evaluate before waiting for user input |

---

## 6.4 Parallel Execution

### Parallel Stages (Declarative)

```groovy
pipeline {
    agent none
    stages {
        stage('Build and Test All Platforms') {
            parallel {
                stage('Linux') {
                    agent { label 'linux' }
                    steps { sh 'make build-and-test' }
                }
                stage('Windows') {
                    agent { label 'windows' }
                    steps { bat 'build-and-test.bat' }
                }
                stage('macOS') {
                    agent { label 'macos' }
                    steps { sh 'make build-and-test' }
                }
            }
        }
    }
}
```

### Parallel with Matrix

```groovy
pipeline {
    agent none
    stages {
        stage('Build All') {
            matrix {
                axes {
                    axis {
                        name 'OS'
                        values 'linux', 'windows', 'macos'
                    }
                    axis {
                        name 'JDK'
                        values '11', '17'
                    }
                }
                agent { label "${OS}" }
                stages {
                    stage('Build') {
                        tools { jdk "JDK ${JDK}" }
                        steps {
                            echo "Building on ${OS} with JDK ${JDK}"
                            sh 'mvn clean compile'
                        }
                    }
                    stage('Test') {
                        steps {
                            sh 'mvn test'
                        }
                    }
                }
            }
        }
    }
}
```

### Parallel Execution with failFast

```groovy
stage('Parallel Tests') {
    failFast true  // Stop all parallel branches if one fails
    parallel {
        stage('Unit Tests') {
            steps { sh 'make unittest' }
        }
        stage('Integration Tests') {
            steps { sh 'make integration' }
        }
        stage('UI Tests') {
            steps { sh 'make uitest' }
        }
    }
}
```

---

## 6.5 Input & Approval Gates

### Stage Input

```groovy
stage('Deploy to Production') {
    input {
        message "Deploy to production?"
        ok "Yes, deploy!"
        submitter "admin,release-managers"
        submitterParameter "APPROVER"
        parameters {
            string(name: 'DEPLOY_VERSION', defaultValue: 'latest')
        }
    }
    steps {
        echo "Approved by: ${APPROVER}"
        sh "deploy.sh --version ${DEPLOY_VERSION}"
    }
}
```

### Timeout for Input

```groovy
stage('Deploy with Timeout') {
    input {
        message "Deploy to production?"
        ok "Deploy"
        timeout(time: 24, unit: 'HOURS')
    }
    steps {
        sh 'deploy.sh production'
    }
}
```

---

## 6.6 Error Handling

### try-catch in Scripted Block

```groovy
stage('Deploy') {
    steps {
        script {
            try {
                sh 'deploy.sh production'
            } catch (Exception e) {
                echo "Deployment failed: ${e.message}"
                currentBuild.result = 'FAILURE'
                // Notify but don't re-throw to continue pipeline
            } finally {
                sh 'cleanup.sh'
            }
        }
    }
}
```

### Retry

```groovy
stage('Flaky Tests') {
    steps {
        retry(3) {
            sh 'make integration-tests'
        }
    }
}
```

### Timeout

```groovy
stage('Long Running Task') {
    steps {
        timeout(time: 30, unit: 'MINUTES') {
            sh 'make slow-build'
        }
    }
}
```

### Using catchError

```groovy
stage('Test') {
    steps {
        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
            sh 'make tests'
        }
    }
    post {
        always {
            junit 'reports/**/*.xml'
        }
    }
}
```

---

## 6.7 File Operations

```groovy
stage('File Operations') {
    steps {
        // Write a file
        writeFile file: 'config.json', text: '{"version": "1.0"}'
        
        // Read a file
        script {
            def config = readFile file: 'config.json'
            echo "Config: ${config}"
        }
        
        // Check if file exists
        script {
            if (fileExists('deploy.sh')) {
                echo "deploy.sh exists"
            }
        }
        
        // Delete a directory
        deleteDir()
        
        // Stash files for later use
        stash name: 'source', includes: 'src/**/*'
    }
}
```

---

## 6.8 Artifacts & Build Reports

```groovy
stage('Collect Artifacts') {
    steps {
        // Archive build artifacts
        archiveArtifacts artifacts: 'target/*.jar',
            excludes: 'target/*-sources.jar',
            fingerprint: true
        
        // Publish JUnit test report
        junit testResults: 'target/surefire-reports/*.xml',
            keepLongStdio: true,
            healthScaleFactor: 0.5
        
        // Publish HTML report
        publishHTML(target: [
            reportName: 'Coverage Report',
            reportDir: 'target/site/coverage',
            reportFiles: 'index.html',
            keepAll: true
        ])
    }
}
```

---

## 6.9 Docker Steps

```groovy
stage('Docker Build') {
    steps {
        script {
            // Build Docker image
            def image = docker.build('my-app:latest', '-f Dockerfile .')
            
            // Push to registry
            docker.withRegistry('https://registry.example.com', 'registry-creds') {
                image.push("${BUILD_NUMBER}")
                image.push('latest')
            }
            
            // Run container for testing
            image.inside('-p 8080:8080') {
                sh 'make test-in-container'
            }
        }
    }
}
```

---

## Key Takeaways

- Use `when` conditions to control stage execution based on branch, expression, or build trigger
- `parallel` executes stages concurrently — use `failFast true` to stop all branches on any failure
- `input` adds manual approval gates before critical stages
- `retry`, `timeout`, and `catchError` help handle failures gracefully
- Stash/unstash pass files between stages on different agents
- `script` wraps arbitrary Groovy code within Declarative pipelines

---

## Next Steps

→ Continue to [Chapter 7: Agents, Nodes & Distributed Builds]({{< relref "07-agents-nodes-and-distributed-builds" >}})
