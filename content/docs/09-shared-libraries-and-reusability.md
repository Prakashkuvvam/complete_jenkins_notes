---
title: "Ch 9: Shared Libraries & Reusability"
weight: 9
---

# Shared Libraries & Reusability

## Learning Objectives

After reading this chapter, you will be able to:
- Create and structure a shared library in Jenkins
- Load libraries globally or per-pipeline
- Write reusable pipeline functions and steps
- Version and test shared libraries
- Implement best practices for library design
- Understand library scoping and security

---

## 9.1 What is a Shared Library?

A **shared library** is a collection of reusable pipeline code stored in a separate Git repository. It allows you to:

- **DRY up pipeline code** — Define common logic once, use everywhere
- **Standardize processes** — Enforce consistent build/deploy patterns
- **Reduce Jenkinsfile size** — Keep pipelines simple and readable
- **Version control pipeline logic** — Library changes can be reviewed like any code

### Library Structure

```
shared-library-repo/
├── vars/                    # Global variables (exposed to pipelines)
│   ├── buildJavaApp.groovy # Define a method
│   ├── deployToK8s.groovy
│   └── slackNotify.groovy
├── src/                     # Groovy classes (helper utilities)
│   └── com/
│       └── company/
│           └── PipelineUtils.groovy
├── resources/               # Non-Groovy files (templates, configs)
│   ├── templates/
│   │   └── deployment.yaml
│   └── config/
│       └── default.properties
└── Jenkinsfile              # Test the library itself
```

---

## 9.2 Creating a Global Variable

A **global variable** is a Groovy file in the `vars/` directory. The filename becomes the step name in pipelines.

### Simple Variable: `vars/sayHello.groovy`

```groovy
// vars/sayHello.groovy
def call(String name = 'World') {
    echo "Hello, ${name}!"
}
```

Usage in pipeline:

```groovy
pipeline {
    agent any
    stages {
        stage('Greet') {
            steps {
                sayHello('Jenkins')    // → "Hello, Jenkins!"
                sayHello()              // → "Hello, World!"
            }
        }
    }
}
```

### More Complex: `vars/buildJavaApp.groovy`

```groovy
// vars/buildJavaApp.groovy
def call(Map params = [:]) {
    def jdkVersion = params.jdkVersion ?: 'JDK 17'
    def mavenVersion = params.mavenVersion ?: 'Maven 3.9'
    def skipTests = params.skipTests ?: false
    def extraArgs = params.extraArgs ?: ''
    
    pipeline {
        agent any
        tools {
            jdk jdkVersion
            maven mavenVersion
        }
        stages {
            stage('Checkout') {
                steps {
                    checkout scm
                }
            }
            stage('Compile') {
                steps {
                    sh "mvn clean compile ${extraArgs}"
                }
            }
            stage('Test') {
                when {
                    expression { !skipTests }
                }
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
                    sh "mvn package -DskipTests ${extraArgs}"
                }
            }
        }
        post {
            success {
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }
    }
}
```

Usage:

```groovy
// Simple usage
buildJavaApp()

// With parameters
buildJavaApp(
    jdkVersion: 'JDK 11',
    skipTests: true,
    extraArgs: '-P production'
)
```

---

## 9.3 Loading & Using Libraries

### Global Libraries

Configured system-wide by Jenkins admin:

Dashboard → Manage Jenkins → Configure System → Global Pipeline Libraries

```
Name: my-shared-lib
Default version: main
Source: Git
Project Repository: https://github.com/org/shared-library.git
Credentials: (optional)
☑ Load implicitly
☑ Allow default version to be overridden
☑ Include in @Library changeset
```

### Per-Pipeline Libraries

```groovy
@Library('my-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Use steps from the library
                buildJavaApp()
            }
        }
    }
}
```

### Multiple Libraries & Versioning

```groovy
// Multiple libraries
@Library(['my-shared-lib@v1.2', 'another-lib@v2.0']) _

// Specific version
@Library('my-shared-lib@feature/new-step') _

// Disable automatic library loading
@Library('my-shared-lib@1.5.0', changelog: false) _
```

---

## 9.4 Using `src/` for Helper Classes

### PipelineUtils.groovy

```groovy
// src/com/company/PipelineUtils.groovy
package com.company

class PipelineUtils implements Serializable {
    private final steps
    
    PipelineUtils(steps) {
        this.steps = steps
    }
    
    def getGitCommitHash() {
        return steps.sh(
            script: 'git rev-parse HEAD',
            returnStdout: true
        ).trim()
    }
    
    def getGitBranch() {
        return steps.sh(
            script: 'git rev-parse --abbrev-ref HEAD',
            returnStdout: true
        ).trim()
    }
    
    def getChangedFiles() {
        def output = steps.sh(
            script: 'git diff --name-only HEAD~1 HEAD',
            returnStdout: true
        ).trim()
        return output ? output.split('\n') : []
    }
    
    def isProductionBranch(String branch) {
        return branch == 'main' || branch ==~ /release\/\d+\.\d+/
    }
    
    def sendSlackNotification(String color, String message) {
        steps.slackSend(
            color: color,
            message: message
        )
    }
    
    def archiveBuildInfo() {
        def info = [
            commit: getGitCommitHash(),
            branch: getGitBranch(),
            buildNumber: steps.env.BUILD_NUMBER,
            timestamp: steps.currentBuild.startTimeInMillis
        ]
        
        def json = steps.writeJSON(
            file: 'build-info.json',
            json: info
        )
        
        steps.archiveArtifacts(
            artifacts: 'build-info.json'
        )
        
        return info
    }
}
```

### Using Helper Classes in Pipeline

```groovy
@Library('my-shared-lib') _
import com.company.PipelineUtils

pipeline {
    agent any
    stages {
        stage('Prepare') {
            steps {
                script {
                    // Create utility instance
                    def utils = new PipelineUtils(this)
                    
                    // Use helper methods
                    def commit = utils.getGitCommitHash()
                    def branch = utils.getGitBranch()
                    
                    echo "Building ${branch} commit ${commit}"
                    
                    if (utils.isProductionBranch(branch)) {
                        echo "Production branch detected!"
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                buildJavaApp()
            }
        }
        
        stage('Archive') {
            steps {
                script {
                    def utils = new PipelineUtils(this)
                    utils.archiveBuildInfo()
                }
            }
        }
    }
}
```

---

## 9.5 Reading Resources

The `resources/` directory holds non-Groovy files accessible via `libraryResource`:

```groovy
// resources/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${APP_NAME}
  labels:
    app: ${APP_NAME}
    version: ${VERSION}
spec:
  replicas: ${REPLICAS}
  selector:
    matchLabels:
      app: ${APP_NAME}
  template:
    metadata:
      labels:
        app: ${APP_NAME}
    spec:
      containers:
      - name: app
        image: ${IMAGE}:${VERSION}
        ports:
        - containerPort: ${PORT}
```

### Using Resources in Pipeline

```groovy
@Library('my-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Generate K8s Manifest') {
            steps {
                script {
                    def template = libraryResource 'templates/deployment.yaml'
                    
                    // Substitute variables
                    def manifest = template
                        .replace('${APP_NAME}', env.APP_NAME)
                        .replace('${VERSION}', env.BUILD_NUMBER)
                        .replace('${IMAGE}', 'myapp:latest')
                        .replace('${REPLICAS}', '3')
                        .replace('${PORT}', '8080')
                    
                    writeFile file: 'deployment.yaml', text: manifest
                }
            }
        }
    }
}
```

---

## 9.6 Library Versioning Strategy

### Semantic Versioning

```
v1.0.0 — Major release (breaking changes)
v1.1.0 — Minor release (new features, backward compatible)
v1.1.1 — Patch release (bug fixes)
```

### Branch-Based Development

```
main          ← v1.2.0 (latest stable)
feature/      ← New features in development
v1.1.x        ← Maintenance branch for v1.1
```

### Referencing Versions

```groovy
// Pin to exact version (recommended for production)
@Library('my-shared-lib@v1.2.0') _

// Use latest from branch (development/testing)
@Library('my-shared-lib@main') _

// Use release tag
@Library('my-shared-lib@v1.0.0') _
```

---

## 9.7 Testing Shared Libraries

### Unit Testing with JenkinsPipelineUnit

```groovy
// test/vars/BuildJavaAppTest.groovy
import com.lesfurets.jenkins.unit.BasePipelineTest
import org.junit.Before
import org.junit.Test

class BuildJavaAppTest extends BasePipelineTest {
    
    @Override
    @Before
    void setUp() throws Exception {
        super.setUp()
        helper.registerAllowedMethod('sh', [Map], { args ->
            println "sh: ${args.script}"
            return ''
        })
        helper.registerAllowedMethod('checkout', [Map], { args -> 
            println "checkout: ${args}"
            return [:]
        })
    }
    
    @Test
    void testBuildWithDefaultParams() throws Exception {
        def script = loadScript('vars/buildJavaApp.groovy')
        script.call([:])
        
        // Assert that specific steps were called
        printCallStack()
        assertJobStatusSuccess()
    }
    
    @Test
    void testBuildWithSkipTests() throws Exception {
        def script = loadScript('vars/buildJavaApp.groovy')
        script.call(skipTests: true)
        
        printCallStack()
        assertJobStatusSuccess()
    }
}
```

### Integration Testing

```groovy
// Jenkinsfile.test
@Library('my-shared-lib@PR-42') _

pipeline {
    agent any
    stages {
        stage('Test Library') {
            steps {
                // Test each step in the library
                sayHello('Test')
                
                buildJavaApp(
                    skipTests: true,
                    extraArgs: '-q'
                )
            }
        }
    }
}
```

---

## 9.8 Best Practices

### Do's

| Practice | Reason |
|----------|--------|
| Keep libraries focused | One library per concern (build, deploy, notify) |
| Serialize everything | All classes should implement `Serializable` |
| Pin versions | Always specify a version in production pipelines |
| Document APIs | Add comments to every public variable/function |
| Write unit tests | Use JenkinsPipelineUnit for library testing |
| Use `@NonCPS` sparingly | Only for methods that can't be serialized |
| Validate parameters | Check required params with defaults or errors |

### Don'ts

| Anti-Pattern | Why |
|------------|-----|
| Global variables calling other global variables | Hard to test and debug |
| Using `input` inside a library | Blocks all pipelines using the library |
| Hard-coding credentials or URLs | Makes library non-portable |
| Loading large files into memory | Can cause OOM errors on the master |
| Ignoring serialization | Pipeline may fail with NotSerializableException |

---

## Key Takeaways

- Shared libraries store reusable pipeline steps in a separate Git repo
- `vars/` defines global variables (filename = step name)
- `src/` contains Groovy classes for complex helper logic
- `resources/` stores templates and configuration files
- Version libraries with tags for safe consumption across pipelines
- Test libraries with JenkinsPipelineUnit and pin versions in production
- Always implement `Serializable` in library classes

---

## Next Steps

→ Continue to [Chapter 10: Multi-Branch Pipelines]({{< relref "10-multi-branch-pipelines" >}})
