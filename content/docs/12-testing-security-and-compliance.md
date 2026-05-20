---
title: "Ch 12: Testing, Security & Compliance"
weight: 12
---

# Testing, Security & Compliance

## Learning Objectives

After reading this chapter, you will be able to:
- Integrate automated testing into Jenkins pipelines
- Implement security scanning in CI/CD
- Manage secrets and credentials securely
- Configure role-based access control (RBAC)
- Implement audit logging and compliance tracking
- Use security plugins for vulnerability detection

---

## 12.1 Automated Testing in Pipelines

### Unit Tests

```groovy
pipeline {
    agent any
    stages {
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```

### Integration Tests

```groovy
stage('Integration Tests') {
    steps {
        sh 'mvn verify -Pintegration-test'
    }
    post {
        always {
            junit 'target/failsafe-reports/*.xml'
            publishHTML(target: [
                reportName: 'Integration Report',
                reportDir: 'target/site/failsafe-report',
                reportFiles: 'index.html'
            ])
        }
    }
}
```

### Code Quality Analysis

```groovy
stage('Code Quality') {
    tools {
        maven 'Maven 3.9'
    }
    steps {
        sh 'mvn checkstyle:checkstyle pmd:pmd spotbugs:spotbugs'
    }
    post {
        always {
            // Publish reports
            checkstyle pattern: 'target/checkstyle-result.xml'
            pmd pattern: 'target/pmd.xml'
            spotbugs pattern: 'target/spotbugsXml.xml'
            
            // Quality gates
            recordIssues(
                tools: [checkStyle(), pmdParser(), spotBugs()],
                qualityGates: [
                    [threshold: 1, type: 'NEW', unstable: true],
                    [threshold: 100, type: 'TOTAL', severity: 'WARNING']
                ]
            )
        }
    }
}
```

---

## 12.2 Security Scanning

### SAST (Static Analysis)

```groovy
stage('Security Scan - SAST') {
    steps {
        script {
            // Using SpotBugs for Java
            sh 'mvn com.github.spotbugs:spotbugs-maven-plugin:spotbugs'
            
            // Using Trivy for filesystem scan
            sh '''
                trivy filesystem --severity HIGH,CRITICAL --exit-code 0 \\
                    --format sarif --output trivy-fs.sarif .
            '''
        }
    }
    post {
        always {
            publishIssues(
                issues: [scanForIssues(tool: spotBugs())],
                qualityGates: [
                    [threshold: 1, type: 'NEW', severity: 'HIGH', unstable: true]
                ]
            )
        }
    }
}
```

### Dependency Scanning

```groovy
stage('Dependency Check') {
    steps {
        script {
            // OWASP Dependency Check
            dependencyCheck(
                odcInstallation: 'OWASP-DC',
                path: '.',
                formats: ['HTML', 'JSON'],
                failBuildOnCVSS: 7,
                suppressionFile: 'dependency-check-suppression.xml'
            )
        }
    }
    post {
        always {
            dependencyCheckPublisher(
                pattern: 'dependency-check-report.xml'
            )
        }
    }
}
```

### Container Scanning

```groovy
stage('Container Security') {
    steps {
        script {
            docker.build('myapp:latest')
            
            // Scan with Trivy
            sh '''
                trivy image --severity HIGH,CRITICAL \\
                    --exit-code 1 \\
                    --format sarif \\
                    --output container-scan.sarif \\
                    myapp:latest
            '''
        }
    }
}
```

---

## 12.3 Secrets Management

### Using withCredentials

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                withCredentials([
                    // Secret text (API keys, tokens)
                    string(credentialsId: 'api-key', variable: 'API_KEY'),
                    
                    // Username/password
                    usernamePassword(
                        credentialsId: 'db-creds',
                        usernameVariable: 'DB_USER',
                        passwordVariable: 'DB_PASS'
                    ),
                    
                    // SSH key
                    sshUserPrivateKey(
                        credentialsId: 'deploy-key',
                        keyFileVariable: 'SSH_KEY',
                        passphraseVariable: 'SSH_PASSPHRASE',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        # Use credentials securely
                        deploy.sh \\
                            --api-key "$API_KEY" \\
                            --db-host "$DB_HOST" \\
                            --db-user "$DB_USER" \\
                            --db-pass "$DB_PASS"
                    '''
                }
            }
        }
    }
}
```

### Masking Secrets in Logs

```groovy
stage('Mask Secret') {
    steps {
        script {
            // Wrap commands that output secrets
            wrap([$class: 'MaskPasswordsBuildWrapper',
                varPasswordPairs: [
                    password: 'my-secret-api-key-12345'
                ]
            ]) {
                sh '''
                    # The secret value will be masked in logs
                    echo "Using API key: my-secret-api-key-12345"
                    # Output: Using API key: ****
                '''
            }
        }
    }
}
```

### HashiCorp Vault Integration

```groovy
stage('Vault Secrets') {
    steps {
        withVault(
            configuration: [
                vaultUrl: 'https://vault.example.com',
                vaultCredentialId: 'vault-token',
                engineVersion: 2
            ],
            vaultSecrets: [
                [
                    path: 'secret/data/myapp/prod',
                    engineVersion: 2,
                    secretValues: [
                        [envVar: 'DB_PASSWORD', vaultKey: 'db_password'],
                        [envVar: 'API_KEY', vaultKey: 'api_key']
                    ]
                ]
            ]
        ) {
            sh '''
                echo "Using DB from Vault"
                deploy.sh --db-pass "$DB_PASSWORD"
            '''
        }
    }
}
```

---

## 12.4 Credential Management Best Practices

| Practice | Description |
|----------|-------------|
| **Least privilege** | Grant only the permissions needed |
| **Rotate regularly** | Change credentials on schedule or on breach |
| **Scope credentials** | Use folder-level credentials, not global |
| **Audit credential usage** | Monitor which jobs use which credentials |
| **Never hard-code** | Always use credentialsId in Jenkins |
| **Use short-lived tokens** | Prefer temporary tokens over long-lived keys |
| **Encrypt at rest** | Jenkins encrypts credentials in config files |

---

## 12.5 Role-Based Access Control (RBAC)

### Global Roles

| Role | Permissions |
|------|-------------|
| **admin** | Full system access |
| **developer** | Build jobs, view system, view agents |
| **viewer** | Read-only access to everything |
| **operator** | Build jobs, view agents |

### Project Roles

```groovy
// Project roles limit access to specific jobs/folders

Role: team-alpha-developer
Pattern: team-alpha/*
Permissions:
  - Job: Build, Read, Discover
  - Run: Update, Delete
  - SCML: Tag
  - Credentials: View

Role: team-alpha-viewer
Pattern: team-alpha/*
Permissions:
  - Job: Read
  - Run: View
```

### Implementing RBAC

1. Install **Role-based Authorization Strategy** plugin
2. Dashboard → Manage Jenkins → Configure Global Security
3. Authorization: **Role-Based Strategy**
4. Manage and Assign Roles:

```
Manage Roles:
  Global roles:
    - admin:     All permissions
    - engineer:  Overall.Read, Job.*, Run.*
    - viewer:    Overall.Read, Job.Read, Run.View

  Project roles:
    - team-alpha:   Pattern: team-alpha/*
                    Permissions: Job.Build, Job.Read, Run.View
    
    - prod-deploy:  Pattern: */main, */release/*
                    Permissions: Job.Build, Run.Update
```

---

## 12.6 Audit Logging

### Enable Job History

```groovy
// Jenkins configuration to enable detailed audit
// Dashboard → Manage Jenkins → Configure System

☑ Enable Job History
  History file path: /var/jenkins_home/jobhistory
  ☐ Enable permission-based history events
```

### Audit Trail Plugin

1. Install **Audit Trail** plugin
2. Configure:

```
Logger name: AuditTrail
Log File: /var/log/jenkins/audit.log

☑ Log user authentication events
☑ Log job configuration changes
☑ Log system configuration changes
☑ Log credential changes
☑ Log agent connection/disconnection
☑ Log build queue events
```

### Sample Audit Log

```
2024-01-15 10:23:45 UTC - admin - CONFIGURE - Job 'myapp/main'
2024-01-15 10:23:46 UTC - admin - BUILD - Job 'myapp/main' #1024
2024-01-15 10:30:00 UTC - admin - DELETE - Run 'myapp/main' #1000
2024-01-15 11:00:00 UTC - jenkins - LOGIN - System user login
2024-01-15 11:15:30 UTC - deploy-bot - TRIGGER - Job 'deploy/production'
```

---

## 12.7 Compliance & Governance

### Pipeline Compliance Checks

```groovy
pipeline {
    agent any
    stages {
        stage('Compliance Checks') {
            steps {
                script {
                    // Check branch policies
                    if (env.BRANCH_NAME == 'main') {
                        // Require PR approval
                        def hasApproval = sh(
                            script: 'gh pr view --json approvals',
                            returnStdout: true
                        ).contains('"approvals"')
                        
                        if (!hasApproval) {
                            error("Main branch requires PR approval")
                        }
                    }
                    
                    // Check signed commits
                    def commits = sh(
                        script: 'git log -1 --pretty=format:"%G?"',
                        returnStdout: true
                    ).trim()
                    
                    if (commits != 'G') { // G = Good signature
                        error("Commit must be signed with a verified GPG key")
                    }
                    
                    // Check license headers
                    sh 'mvn license:check'
                }
            }
        }
        
        stage('SBOM Generation') {
            steps {
                sh '''
                    # Generate Software Bill of Materials
                    mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom
                '''
                post {
                    always {
                        archiveArtifacts artifacts: 'target/**/*.xml'
                    }
                }
            }
        }
    }
}
```

### Compliance Reports

```groovy
stage('Compliance Report') {
    steps {
        script {
            def report = """
Compliance Report - ${env.JOB_NAME} #${env.BUILD_NUMBER}
========================================================
Date: ${new Date().format('yyyy-MM-dd HH:mm:ss')}
Branch: ${env.BRANCH_NAME}
Commit: ${env.GIT_COMMIT}

Checks:
  ✓ PR approval: Required for main
  ✓ Signed commits: Enforced
  ✓ License headers: Verified
  ✓ Dependency scan: ${env.DEPENDENCY_CHECK_RESULT}
  ✓ Container scan: ${env.CONTAINER_SCAN_RESULT}
  ✓ SAST scan: ${env.SAST_RESULT}

Build: ${env.JOB_URL}${env.BUILD_NUMBER}
"""
            writeFile file: 'compliance-report.txt', text: report
            archiveArtifacts artifacts: 'compliance-report.txt'
        }
    }
}
```

---

## Key Takeaways

- Integrate unit, integration, and security tests at every pipeline stage
- Use `withCredentials` for secure secret injection — never hard-code secrets
- SAST, dependency scanning, and container scanning catch vulnerabilities early
- RBAC with folder-level patterns enforces least privilege across teams
- Audit logging provides traceability for compliance requirements
- Generate SBOM and compliance reports for regulatory needs

---

## Next Steps

→ Continue to [Chapter 13: JCasC — Configuration as Code]({{< relref "13-jenkins-configuration-as-code" >}})
