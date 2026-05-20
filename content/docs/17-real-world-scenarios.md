---
title: "Ch 17: Real-World Scenario Questions"
weight: 17
---

# Real-World Scenario Questions

## Learning Objectives

- Apply Jenkins knowledge to practical scenarios
- Troubleshoot real-world CI/CD problems
- Design solutions for common Jenkins challenges
- Understand the trade-offs in architectural decisions

---

## Scenario 1: CI/CD Migration

**Context:** Your team currently builds and deploys manually by SSHing into servers, running build commands, and copying JAR files. Deployments happen once a week and frequently break.

**Task:** Design a CI/CD pipeline that addresses these problems.

**Solution:**

```groovy
pipeline {
    agent any
    triggers {
        githubPush()
    }
    stages {
        stage('Build') {
            steps { sh 'mvn clean package' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
            post { always { junit 'target/surefire-reports/*.xml' } }
        }
        stage('Deploy to Staging') {
            when { branch 'develop' }
            steps { sh 'deploy.sh staging' }
        }
        stage('Deploy to Production') {
            when { branch 'main' }
            input { message "Deploy to production?" }
            steps { sh 'deploy.sh production' }
        }
    }
}
```

**Key improvements:**
- Automated builds on every commit
- Test automation catches bugs early
- Staging environment for validation
- Approval gate for production releases
- Rollback capability by keeping previous artifacts

---

## Scenario 2: CI/CD for Microservices

**Context:** You have 15 microservices in separate repositories. Each service has its own build, test, and deploy pipeline. Teams want to reuse common pipeline logic.

**Task:** Design a scalable Jenkins setup for microservices.

**Solution:**

1. **Single shared library** with common build/deploy steps:

```groovy
// vars/buildMicroservice.groovy
def call(Map config = [:]) {
    pipeline {
        agent any
        parameters {
            string(name: 'SERVICE_NAME', defaultValue: config.serviceName)
        }
        stages {
            stage('Build') {
                steps {
                    sh "cd services/${SERVICE_NAME} && mvn clean package"
                }
            }
            stage('Test') {
                steps { sh "cd services/${SERVICE_NAME} && mvn test" }
            }
            stage('Dockerize') {
                steps {
                    sh "docker build -t ${SERVICE_NAME}:latest services/${SERVICE_NAME}"
                }
            }
        }
    }
}
```

2. **Organization folder** to discover all service repos
3. **Multi-Branch Pipeline** configuration per repo
4. **One Jenkinsfile per service** that calls the shared library:

```groovy
@Library('ms-common-lib') _
buildMicroservice(serviceName: 'user-service')
```

---

## Scenario 3: Database Migration in Pipeline

**Context:** You need to run database migrations as part of your pipeline. The migration script requires database credentials and must run only once per deployment.

**Task:** Implement safe database migrations in a pipeline.

**Solution:**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn clean package' }
        }
        stage('Migrate Database') {
            when { branch 'main' }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'db-credentials',
                    usernameVariable: 'DB_USER',
                    passwordVariable: 'DB_PASSWORD'
                )]) {
                    sh '''
                        # Run migrations with idempotency check
                        migrate.sh \\
                            --url jdbc:postgresql://db.example.com:5432/appdb \\
                            --user "$DB_USER" \\
                            --password "$DB_PASSWORD" \\
                            --locations filesystem:migrations
                    '''
                }
            }
        }
        stage('Deploy') {
            steps { sh 'deploy.sh' }
        }
    }
}
```

**Safety measures:**
- Run migrations BEFORE deploying the new code
- Use idempotent migration scripts (Flyway/Liquibase)
- Always have a rollback migration ready
- Test migrations in staging first

---

## Scenario 4: Container Security Scan

**Context:** Security team requires all container images to be scanned for vulnerabilities before deployment. High-severity CVEs must block the build.

**Task:** Integrate container security scanning into the pipeline.

**Solution:**

```groovy
pipeline {
    agent any
    environment {
        IMAGE = "myapp:${BUILD_NUMBER}"
    }
    stages {
        stage('Build') {
            steps { sh 'mvn clean package' }
        }
        stage('Docker Build') {
            steps { sh 'docker build -t $IMAGE .' }
        }
        stage('Security Scan') {
            steps {
                sh '''
                    trivy image --severity HIGH,CRITICAL \\
                        --exit-code 1 $IMAGE
                '''
            }
            post {
                failure {
                    // Allow override for false positives
                    input message: "Scan failed. Override?",
                        submitter: "security-team"
                }
            }
        }
        stage('Push & Deploy') {
            steps {
                sh 'docker push $IMAGE'
                sh 'kubectl set image deployment/myapp myapp=$IMAGE'
            }
        }
    }
}
```

---

## Scenario 5: Multi-Environment Promotion

**Context:** An application needs to be promoted through Dev → Staging → Production environments. Each environment requires different configurations and approvals.

**Task:** Design a promotion pipeline.

**Solution:**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn clean package' }
            post { success { archiveArtifacts 'target/*.jar' } }
        }
        
        stage('Dev Deploy') {
            when { branch 'develop' }
            steps { sh 'deploy.sh dev' }
        }
        
        stage('Staging Promote') {
            when { branch 'release/*' }
            steps { sh 'deploy.sh staging' }
        }
        
        stage('Staging Tests') {
            steps { sh 'make integration-tests' }
        }
        
        stage('Production Approval') {
            when { branch 'main' }
            input {
                message "Promote to production?"
                submitter "release-manager"
                parameters {
                    string(name: 'VERSION', 
                        defaultValue: '', 
                        description: 'Version from staging')
                }
            }
        }
        
        stage('Production Deploy') {
            when { branch 'main' }
            steps {
                sh "deploy.sh production --version ${params.VERSION}"
            }
        }
    }
}
```

---

## Scenario 6: Pipeline Failure Recovery

**Context:** A multi-hour integration test pipeline fails midway due to a transient infrastructure issue. Developers need to resume from the failure point without rebuilding everything.

**Task:** Design a resilient pipeline with failure recovery.

**Solution:**

```groovy
pipeline {
    agent any
    options {
        preserveStashes(buildCount: 10)
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
                stash name: 'build-output', includes: 'target/*.jar'
            }
        }
        
        stage('Unit Tests') {
            steps {
                unstash 'build-output'
                sh 'mvn test'
            }
        }
        
        stage('Integration Tests') {
            steps {
                unstash 'build-output'
                retry(3) {
                    timeout(30) {
                        sh 'mvn verify -Pintegration'
                    }
                }
            }
        }
        
        stage('Performance Tests') {
            when {
                expression { params.RUN_PERF_TESTS }
            }
            steps {
                unstash 'build-output'
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh 'make perf-tests'
                }
            }
        }
    }
}
```

**Resilience features:**
- `retry(3)` for transient failures
- `timeout` to prevent hung builds
- `catchError` for non-critical stages
- `preserveStashes` for debugging
- Parallel execution with `failFast false`

---

## Scenario 7: Secrets Rotation

**Context:** Security policy requires all Jenkins credentials to be rotated every 90 days. You have 200+ credentials across 30 Jenkins folders.

**Task:** Automate credential rotation.

**Solution:**

```groovy
// Rotation script (run as a pipeline job)
pipeline {
    agent any
    stages {
        stage('Generate New Secrets') {
            steps {
                script {
                    // Generate new API keys from Vault
                    def newKeys = sh(
                        script: '''
                            vault write -format=json aws/creds/jenkins |
                            jq '{access_key: .data.access_key, 
                                 secret_key: .data.secret_key}'
                        ''',
                        returnStdout: true
                    ).trim()
                    
                    writeFile file: 'new-creds.json', text: newKeys
                }
            }
        }
        
        stage('Update Jenkins Credentials') {
            steps {
                script {
                    def creds = readJSON file: 'new-creds.json'
                    
                    // Use Jenkins API to update credentials
                    sh """
                        curl -X POST http://localhost:8080/credentials/store/system/domain/_/updateCredentials \\
                            --user admin:\${ADMIN_TOKEN} \\
                            --data-urlencode 'json={
                                "credentials": {
                                    "scope": "GLOBAL",
                                    "id": "aws-jenkins-role",
                                    "username": "${creds.access_key}",
                                    "password": "${creds.secret_key}",
                                    "description": "Rotated on ${new Date()}",
                                    "\$class": "com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl"
                                }
                            }'
                    """
                }
            }
        }
    }
}
```

**Best practices:**
- Use Vault for dynamic secret generation
- Schedule rotation jobs with cron
- Validate new credentials before deleting old ones
- Keep a window of overlapping validity
- Notify teams of upcoming rotations

---

## Scenario 8: Large-Scale Monorepo

**Context:** A monorepo contains 50+ projects in different languages (Java, Node, Python). Building everything on every commit takes 2+ hours. Teams want to build only what changed.

**Task:** Design an optimized monorepo pipeline.

**Solution:**

```groovy
pipeline {
    agent any
    stages {
        stage('Detect Changes') {
            steps {
                script {
                    // Get changed files
                    def changedFiles = sh(
                        script: 'git diff --name-only HEAD~1 HEAD',
                        returnStdout: true
                    ).trim().split('\n')
                    
                    // Determine which projects changed
                    def projectsToBuild = []
                    changedFiles.each { file ->
                        if (file.startsWith('services/java/')) {
                            projectsToBuild.add('java')
                        } else if (file.startsWith('services/node/')) {
                            projectsToBuild.add('node')
                        } else if (file.startsWith('services/python/')) {
                            projectsToBuild.add('python')
                        }
                    }
                    
                    env.BUILD_JAVA = projectsToBuild.contains('java') ? 'true' : 'false'
                    env.BUILD_NODE = projectsToBuild.contains('node') ? 'true' : 'false'
                    env.BUILD_PYTHON = projectsToBuild.contains('python') ? 'true' : 'false'
                }
            }
        }
        
        stage('Build Changed Projects') {
            parallel {
                stage('Java') {
                    when { expression { env.BUILD_JAVA == 'true' } }
                    steps {
                        sh 'cd services/java && mvn clean package'
                    }
                }
                stage('Node') {
                    when { expression { env.BUILD_NODE == 'true' } }
                    steps {
                        sh 'cd services/node && npm ci && npm run build'
                    }
                }
                stage('Python') {
                    when { expression { env.BUILD_PYTHON == 'true' } }
                    steps {
                        sh 'cd services/python && pip install -r requirements.txt'
                    }
                }
            }
        }
    }
}
```

---

## Scenario 9: Zero-Downtime Deployment

**Context:** A critical web application cannot tolerate downtime during deployment. You need to deploy new versions without interrupting user traffic.

**Task:** Implement a zero-downtime deployment strategy.

**Solution:**

```groovy
pipeline {
    agent any
    environment {
        APP_NAME = 'myapp'
        K8S_NAMESPACE = 'production'
    }
    stages {
        stage('Build & Push Image') {
            steps {
                script {
                    docker.build("${APP_NAME}:${BUILD_NUMBER}").push()
                }
            }
        }
        
        stage('Rolling Update') {
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \\
                        ${APP_NAME}=${APP_NAME}:${BUILD_NUMBER} \\
                        -n ${K8S_NAMESPACE} \\
                        --record
                    
                    # Wait for rollout to complete
                    kubectl rollout status deployment/${APP_NAME} \\
                        -n ${K8S_NAMESPACE} \\
                        --timeout=10m
                """
            }
        }
        
        stage('Smoke Test') {
            steps {
                sh '''
                    # Verify the new deployment
                    URL=$(kubectl get svc ${APP_NAME} -n ${K8S_NAMESPACE} \\
                        -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                    curl -f http://$URL/health
                '''
            }
        }
    }
    post {
        failure {
            // Rollback on failure
            sh "kubectl rollout undo deployment/${APP_NAME} -n ${K8S_NAMESPACE}"
        }
    }
}
```

---

## Scenario 10: Jenkins Pipeline for Infrastructure as Code

**Context:** Your platform team manages AWS infrastructure using Terraform. Changes must be reviewed before apply, and state must be locked to prevent concurrent modifications.

**Task:** Design a Terraform pipeline with plan/apply workflow.

**Solution:**

```groovy
pipeline {
    agent any
    parameters {
        choice(name: 'ACTION', choices: ['plan', 'apply'], description: 'Terraform action')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Terraform Init') {
            steps {
                sh 'terraform init -backend-config=backend-prod.hcl'
            }
        }
        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
            post {
                success {
                    // Archive plan for apply
                    stash name: 'tfplan', includes: 'tfplan'
                }
            }
        }
        stage('Terraform Apply') {
            when {
                expression { params.ACTION == 'apply' }
                branch 'main'
            }
            input {
                message "Apply this Terraform plan?"
                ok "Apply"
            }
            steps {
                unstash 'tfplan'
                sh 'terraform apply tfplan'
            }
        }
    }
}
```

---

## Scenario 11: Emergency Hotfix Pipeline

**Context:** A critical production bug needs an immediate fix. The normal CI/CD pipeline takes 2 hours with full test suites. The hotfix must bypass some steps but still maintain quality.

**Task:** Design an emergency hotfix pipeline with guardrails.

**Solution:**

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'HOTFIX_BRANCH', 
            description: 'Branch name for the hotfix')
        booleanParam(name: 'SKIP_LONG_TESTS', 
            defaultValue: true,
            description: 'Skip integration and performance tests')
    }
    stages {
        stage('Validate Hotfix') {
            steps {
                script {
                    // Ensure hotfix branch naming convention
                    if (!env.HOTFIX_BRANCH.startsWith('hotfix/')) {
                        error("Branch must start with 'hotfix/'")
                    }
                    
                    // Require JIRA ticket
                    def commitMsg = sh(
                        script: 'git log -1 --pretty=%s',
                        returnStdout: true
                    ).trim()
                    
                    if (!(commitMsg =~ /HOTFIX-\d+/)) {
                        error("Commit must reference a JIRA hotfix ticket")
                    }
                }
            }
        }
        
        stage('Quick Tests') {
            steps {
                sh 'mvn test -DskipITs=true'
            }
            post { always { junit 'target/surefire-reports/*.xml' } }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Skip staging for hotfix
                    sh 'deploy.sh production --hotfix'
                }
            }
        }
        
        stage('Post-Deploy Verification') {
            steps {
                sh 'run-smoke-tests.sh'
            }
        }
    }
    
    post {
        always {
            // Notify team of hotfix deployment
            slackSend(
                color: '#FF0000',
                message: "🚨 HOTFIX ${env.HOTFIX_BRANCH} deployed to production"
            )
        }
    }
}
```

---

## Scenario 12: Compliance and Audit Pipeline

**Context:** A financial services company needs every build to produce an auditable trail showing who approved what, when, and what code was deployed.

**Task:** Design a compliance-focused pipeline.

**Solution:**

```groovy
pipeline {
    agent any
    stages {
        stage('Pre-Build Compliance') {
            steps {
                script {
                    // Verify GPG-signed commit
                    def signature = sh(
                        script: 'git log -1 --pretty=format:"%G?"',
                        returnStdout: true
                    ).trim()
                    
                    if (signature != 'G') {
                        error("Commit must be signed with a verified GPG key")
                    }
                    
                    // Record who triggered the build
                    echo "Build triggered by: ${env.BUILD_USER}"
                    echo "Build user ID: ${env.BUILD_USER_ID}"
                }
            }
        }
        
        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
        
        stage('Approval Gate') {
            when { branch 'main' }
            input {
                message "Approve deployment?"
                submitter "compliance-officer,release-manager"
                submitterParameter "APPROVED_BY"
            }
            steps {
                script {
                    echo "Approved by: ${APPROVED_BY}"
                }
            }
        }
        
        stage('Generate Audit Report') {
            steps {
                script {
                    def report = """
Audit Report - ${env.JOB_NAME} #${env.BUILD_NUMBER}
======================================================
Date: ${new Date().format('yyyy-MM-dd HH:mm:ss')}
Triggered by: ${env.BUILD_USER} (${env.BUILD_USER_ID})
Commit: ${env.GIT_COMMIT}
Branch: ${env.BRANCH_NAME}
Approved by: ${APPROVED_BY ?: 'N/A'}
Build result: ${currentBuild.result ?: 'PENDING'}
======================================================
"""
                    writeFile file: 'audit-report.txt', text: report
                    archiveArtifacts artifacts: 'audit-report.txt'
                }
            }
        }
    }
}
```

---

## Key Takeaways

- Design pipelines for resilience — use retries, timeouts, and stash/unstash
- Implement approval gates for production deployments
- Use shared libraries for consistency across microservices
- Automate credential rotation with Vault and Jenkins API
- Zero-downtime deployments require rolling updates and health checks
- Compliance pipelines must log who did what and when
- Emergency hotfix pipelines need guardrails, not full bypass

---

## Next Steps

→ [Take Practice Test 1]({{< relref "18-exam-practice-test-1" >}}) to test your knowledge
