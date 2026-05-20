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

### Detailed Explanation

This pipeline addresses the core problems of manual deployments: human error, inconsistency, and lack of traceability. Let's break down why each piece matters:

**Why automated builds?** Manual builds are error-prone because engineers forget steps, use different tool versions, or build on differently-configured machines. By putting build steps in a Jenkinsfile, every build runs identically every time — this is the foundation of "it works on my machine" prevention.

**Why tests before deploy?** The `junit` post-action collects test reports so Jenkins can track trends over time. If test pass rates drop, you'll see it on the Jenkins dashboard before it reaches production. The `post { always }` block ensures test results are captured even if later stages fail.

**Why staging then production?** Staging serves as a gate — it validates that the artifact works in an environment similar to production. The `when { branch 'develop' }` condition auto-deploys to staging, while production requires `branch 'main'` AND manual approval via the `input` directive. This two-tier approach catches ~80% of issues in staging before they ever reach prod.

**Trade-offs to consider:**
- This pipeline builds on every commit — for large teams, you may want to throttle builds or use "build on demand" triggers
- The `input` step blocks the pipeline until someone approves — if no one approves for hours, the deployment queue backs up. Consider adding a timeout: `input(timeout: 60, message: "Deploy?")`
- No rollback mechanism is shown — in practice, keep the last 5 successful JARs archived and add a "Rollback" stage that redeploys a previous version
- Deployment scripts (`deploy.sh`) are assumed to exist — these should be idempotent (safe to run multiple times)

**Production enhancement:** Add a notification stage that posts build status to Slack or email, so the team knows when a deployment is waiting for approval.

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

### Detailed Explanation

This architecture solves two fundamental challenges: **consistency** across 15 services and **maintainability** of 15 pipelines. Here's why each component matters:

**Why a shared library?** Without a shared library, each of the 15 microservices would duplicate the same pipeline logic. When you need to add a security scan stage or update a build tool version, you'd have to edit 15 Jenkinsfiles — a maintenance nightmare. The shared library centralizes this logic. The `def call(Map config = [:])` pattern makes the library a function that services can call with different parameters (like `serviceName`), so each service gets its own unique configuration while sharing the same pipeline structure.

**Why Organization Folders?** Organization Folders (or GitHub Organization folders) automatically discover all repositories in your GitHub/GitLab org. When a new microservice is added, Jenkins auto-detects it — no manual job creation needed. This is critical at scale because manually creating 15+ jobs is error-prone and doesn't scale to 50+ services.

**Why Multi-Branch Pipelines?** Multi-Branch Pipelines automatically create a pipeline for each branch in a repo (main, develop, feature/*, etc.) and trigger builds on push/PR. This gives developers immediate feedback on their feature branches before merging.

**Why one Jenkinsfile per service?** Each service needs to specify its own unique configuration (service name, test paths, deployment targets). The Jenkinsfile is minimal — just a `@Library` annotation and a single function call — making it easy for each team to own their pipeline without touching shared infrastructure.

**Trade-offs to consider:**
- Shared library changes affect ALL services — version your library with Git tags and use `@Library('ms-common-lib@v2.1')` to pin versions
- If services have vastly different build processes, the shared abstraction may become too complex with too many parameters — in that case, group services by build type (Java services, Node services) with separate shared libraries
- Organization Folders can be slow to scan if you have hundreds of repos — adjust the scan interval or trigger scans via webhooks
- The `parameters` block in `buildMicroservice.groovy` shows a `SERVICE_NAME` parameter — consider deriving this from the repo name automatically with `env.REPO_NAME` to reduce manual inputs

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

### Detailed Explanation

Database migrations are one of the riskiest operations in a deployment pipeline — a failed migration can corrupt data, cause downtime, or require a time-consuming manual recovery. Here's why this design is safe:

**Why run migrations BEFORE deploying code?** This is the most important ordering constraint. If you deploy the new code first, it may immediately try to query the old schema and crash. If you migrate first, both old and new code can safely coexist (the old code simply ignores the new column/table). This ordering matches the "expand-migrate-contract" pattern: expand the schema, migrate data, then deploy code that uses the new schema.

**Why `withCredentials`?** Database credentials should never be hardcoded in pipeline code or stored in environment variables visible in the Jenkins console log. `withCredentials` injects environment variables securely — they're masked in logs and only available within the block. The credentials should be stored in Jenkins' credential store with a unique ID (`db-credentials`).

**Why use Flyway/Liquibase?** These tools track which migrations have already been applied using a tracking table (e.g., `flyway_schema_history`). Running `migrate.sh` a second time only applies new migrations — this idempotency is critical because builds can be retried.

**Why the `when { branch 'main' }` condition?** Database migrations should only run against shared environments (staging, production), not feature branches. Feature branches typically don't have their own databases, and if they did, you'd want separate migration scripts for each PR.

**Rollback strategy:** Always create a paired rollback migration (e.g., `V2_1__add_users_table.sql` and `U2_1__remove_users_table.sql`). If the deployment fails after migration, run the rollback before redeploying the old code. Without a rollback, you'd need to manually restore from backup — which could cost hours of downtime.

**Common pitfalls:**
- Running migrations on read replicas instead of the primary database
- Migrations that take longer than the Jenkins job timeout (add `timeout` to the stage)
- Migrations that lock tables for too long — test migration speed in staging first
- Not testing the rollback migration in staging before production

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

### Detailed Explanation

Container security scanning is a critical gate in modern pipelines — especially with supply chain attacks on the rise. Here's a deeper look at the design decisions:

**Why scan at the image level, not the source level?** Scanning the built container image captures the complete attack surface: the application code, all dependencies (JARs, npm packages), the base OS packages, and any misconfigurations in the Dockerfile. Source-level scanning (SAST) is complementary but doesn't catch vulnerabilities in the OS packages from the base image or in third-party dependencies after they're bundled.

**Why Trivy?** Trivy is chosen because it's open source, fast, comprehensive (scans OS packages, language-specific packages, IaC files), and supports `--exit-code 1` which makes it natively compatible with CI/CD pipelines. Alternatives include:
- **Clair** — good but requires a separate registry setup
- **Anchore Grype** — similar to Trivy but slower initial scan
- **Snyk** — commercial, better for dev workflows but has usage limits
- **Docker Scout** — built into Docker but less configurable

**Why `--severity HIGH,CRITICAL`?** LOW and MEDIUM vulnerabilities are too numerous to block builds on (often thousands per image). Filtering to HIGH and CRITICAL catches the truly dangerous issues while keeping the pipeline moving. Many teams also set a threshold (e.g., "fail if more than 5 HIGH CVEs") rather than failing on any single CVE.

**Why the security-team override?** False positives happen — a vulnerability might be in a package that isn't actually loaded at runtime, or the CVE might not apply to your specific usage. The `input` step with `submitter: "security-team"` allows security engineers to override the block after investigation, rather than forcing developers to rebuild and rescan. This keeps the pipeline moving while maintaining accountability.

**Production hardening:**
- Add a **dependency scan** stage before the Docker build (run `mvn dependency-check:check` or `npm audit`) to catch vulnerabilities earlier — failing early saves build time
- Use a **base image allowlist** — only allow images from approved registries (e.g., your private registry with hardened base images)
- **Sign your images** with Cosign after scanning — so deployment can verify the image was scanned and approved
- Scan the **final running container** too — runtime scanning catches vulnerabilities introduced after image creation (e.g., via volume mounts)

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

### Detailed Explanation

Multi-environment promotion is a core pattern for organizations that need release confidence. Here's the reasoning behind this design:

**Branch-based promotion strategy:** The pipeline uses Git branches as the promotion mechanism — `develop` → `release/*` → `main`. This pattern (GitFlow-inspired) maps each environment to a branch, making the promotion explicit and visible. The alternative is artifact promotion (promoting a specific build number through environments without re-building), which is faster but requires a robust artifact repository and version tracking.

**Why build once, promote many?** The pipeline builds the artifact in the first stage and archives it. Each subsequent deployment stage uses this same artifact. This ensures the exact same binary that passed tests in dev is what gets deployed to production. If you rebuild at each environment, you risk differences between builds (different dependency resolution times, different build servers).

**Why `when { branch 'release/*' }` for staging?** Release branches are where teams stabilize code before production. Deploying to staging from release branches means staging always reflects "what we're about to release" — not random feature branch experiments. When a release branch is ready, it merges to main for production deployment.

**Why use `input` parameters for the version?** The `VERSION` parameter requires the release manager to explicitly specify which version from staging is being promoted to production. This prevents accidentally deploying an older version (because you promoted the wrong build) and creates an audit trail of exactly what version was approved.

**Trade-offs to consider:**
- **GitFlow complexity:** This branch structure requires discipline. Teams new to GitFlow often struggle with release branch management. A simpler alternative is trunk-based development with feature flags — deploy `main` everywhere but control feature activation via flags
- **Approval bottlenecks:** Requiring a "release-manager" as the sole approver creates a single point of failure. Consider having a backup approver or a rotation schedule
- **Integration test duplication:** Tests run in staging, but what if production has different config (larger database, more traffic)? Consider adding **production canary deployments** after the approval gate
- **Manual version parameter:** The VERSION field is free-text and error-prone. A better approach: have the pipeline auto-detect the version from the release branch name or from a VERSION file in the repo

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

### Detailed Explanation

Long-running pipelines (integration tests, performance tests) that fail midway are a major productivity drain. Engineers waste time rebuilding everything just to get back to the failure point. Here's how this design solves that:

**Why `stash`/`unstash`?** Stashing preserves the build output (compiled JARs, node_modules) between stages, even if those stages run on different agents. If the integration test stage fails, you can trigger a re-run and the pipeline picks up from the stashed artifacts — skipping the 5-minute build. Without stashes, every re-run starts from scratch.

**Why `preserveStashes(buildCount: 10)`?** By default, Jenkins deletes stashes when the pipeline completes. Enabling `preserveStashes` keeps the last 10 builds' stashes, so you can debug a failed build from last week without rebuilding. This is invaluable for troubleshooting intermittent failures.

**Why `retry(3)` with `timeout(30)`?** Integration tests fail for many transient reasons: network timeouts, database connection pool exhaustion, DNS resolution failures. `retry(3)` automatically retries up to 3 times, which catches ~90% of transient failures. The `timeout(30)` prevents a hung test from occupying the agent for hours — it kills the stage after 30 minutes and marks it as failed.

**Why `catchError` with `UNSTABLE`?** Performance tests are important but shouldn't block a release — a 2% regression in performance is worth investigating but not worth halting the entire pipeline. `catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE')` keeps the build going but marks it as UNSTABLE (yellow) instead of FAILED (red). The team can review performance trends separately while still shipping features.

**Advanced recovery patterns:**
- **Checkpoint plugin** — Jenkins offers a Checkpoints plugin that can save and restore entire pipeline state (including workspaces), not just stashed files. This is more powerful but has licensing considerations with CloudBees Jenkins
- **Git commit tagging** — tag successful builds with `git tag build-<number>-ok`, so you can quickly identify "last known good" for rollback
- **Stage-level re-run** — modern Jenkins (Blue Ocean interface) allows re-running individual stages. Combined with stash/unstash, this is the most practical recovery workflow for most teams
- **Parallel retry with backoff** — instead of retry(3) with fixed delay, use `sleep time: Math.pow(2, count), unit: 'SECONDS'` for exponential backoff to avoid overwhelming the test infrastructure

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

### Detailed Explanation

Credential rotation is often overlooked until an audit failure or a security incident. At scale (200+ credentials), manual rotation is impossible. Here's why this approach works:

**Why HashiCorp Vault?** Vault provides dynamic secrets — instead of rotating a static credential, Vault generates short-lived credentials on demand with a configurable TTL (e.g., 1 hour). The rotation script doesn't "rotate" in the traditional sense; it requests new credentials, updates Jenkins, and the old ones expire automatically. This is vastly more secure than rotating static credentials every 90 days because the exposure window of a compromised credential is hours, not months.

**Why use the Jenkins API via `curl`?** Jenkins' Credentials plugin has a REST API for creating, updating, and deleting credentials. The API is preferred over the CLI for automation because it's scriptable and can be called from any pipeline. The API endpoint format is: `{jenkins-url}/credentials/store/{store}/domain/{domain}/updateCredentials` — the example uses `store/system` and `domain/_` (the global domain).

**Why the `\$class` with escaped dollar sign?** The Jenkins API uses the `$class` parameter from the Stapler framework to specify which credential implementation to use. `UsernamePasswordCredentialsImpl` is the most common. The backslash escapes the `$` in Groovy's `sh"""` string to prevent Groovy string interpolation.

**Overlapping validity window:** The most critical safety concern during rotation is timing — if you delete old credentials before the new ones are verified, you risk breaking all pipelines suddenly. Best practice:
1. Create new credentials with a temporary ID (e.g., `aws-jenkins-role-new`)
2. Validate the new credentials by running a test build
3. Update all pipelines to use the new credential ID
4. After a cooldown period (e.g., 24 hours), delete the old credentials

**Production considerations:**
- **HTTP vs HTTPS:** Always use HTTPS with proper certificate validation — the `-k` flag (insecure) should never be used
- **Admin token:** The `ADMIN_TOKEN` should be a Jenkins API token (not a password), stored as a credential in Jenkins itself — otherwise you have a chicken-and-egg problem
- **Rate limiting:** Updating 200+ credentials sequentially could take minutes. Add parallel execution with throttling to avoid overwhelming the Jenkins API
- **Audit logging:** Log each rotation event (credential ID, timestamp, user) to an external system like Splunk or CloudWatch
- **Dry-run mode:** Always have a `--dry-run` flag that shows what would change without making actual API calls

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

### Detailed Explanation

Incremental builds in a monorepo are essential for developer productivity. Here's why this design works and its limitations:

**Why detect changes with `git diff --name-only`?** Comparing the current commit to the previous one (`HEAD~1`) tells you exactly which files changed. By checking the directory prefix, you determine which projects need rebuilding. This avoids the expensive "build everything" approach and reduces pipeline time from 2+ hours to minutes for most commits.

**Why `git diff HEAD~1 HEAD` vs `origin/main...HEAD`?** `HEAD~1` works for simple linear histories but can be unreliable for merge commits (which have two parents). A more robust approach uses `git diff --name-only $GIT_PREVIOUS_COMMIT $GIT_COMMIT` — Jenkins exposes these environment variables automatically, which correctly handles merge commits.

**Why parallel stages?** Building Java, Node, and Python projects in parallel takes advantage of multi-core agents or multiple agents. If each build takes 30 minutes standalone, parallel execution can reduce total time to the max of the three (30 min) instead of the sum (90 min). The `parallel` block in Declarative Pipeline runs all stages simultaneously.

**Why `when { expression }` instead of conditional logic in the parallel block?** Each parallel stage has its own `when` condition that checks the environment variable. This means unaffected projects are simply skipped — their stage shows as "Skipped" in the Blue Ocean UI rather than erroring out or requiring complex if/else logic.

**Limitations and improvements:**
- **Change detection is heuristic:** A change to a shared library (`services/shared/java-lib/`) would affect all Java services but might only trigger a Java build for one. Consider using dependency graph tools like **Bazel**, **Nx**, or **TurboRepo** for true dependency-aware builds
- **File-watching approach:** Instead of `git diff`, use tools like **watchman** or **inotify** to trigger builds only when specific files actually change — more efficient than polling git on every commit
- **Docker layer caching:** For monorepos with Docker, cache Docker layers between builds. A change to one Java service shouldn't require downloading Maven dependencies for all Java services
- **Build matrix plugin:** Jenkins' Build Matrix plugin or **Parallel Test Executor** can split test execution across multiple agents for even faster feedback
- **Pre-commit hooks:** Add pre-commit hooks that run linters and unit tests only on changed files before the commit even reaches Jenkins — catching issues earlier

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

### Detailed Explanation

Zero-downtime deployment is a requirement for any customer-facing application. Here's the reasoning behind the Kubernetes rolling update approach:

**Why rolling updates?** Kubernetes rolling updates replace pods one at a time — the service's load balancer directs traffic away from a pod before terminating it, then directs traffic to the new pod once it's healthy. At any point during the deployment, both old and new pods are serving traffic, so users never see a "service unavailable" error. This is the simplest zero-downtime strategy and is built into Kubernetes natively.

**Alternative zero-downtime strategies:**
- **Blue/Green deployment** — run two identical environments (blue = current, green = new). Switch the load balancer from blue to green atomically. Advantage: instant rollback (switch back to blue). Disadvantage: requires 2x infrastructure cost during deployment.
- **Canary deployment** — route 5% of traffic to the new version, monitor for errors, then gradually increase to 100%. Advantage: minimizes blast radius. Disadvantage: complex traffic routing and monitoring setup.
- **Feature flags** — deploy new code but hide the feature behind a flag. Enable the flag for internal users first, then gradually for all users. Advantage: no deployment needed for rollback — just toggle the flag off. Disadvantage: flag management complexity.

**Why `kubectl set image` vs a full deployment manifest?** `kubectl set image` directly updates the container image of a running deployment. It's quick and works well for urgent deployments. However, for production, prefer updating a deployment YAML in Git (GitOps pattern) and using `kubectl apply` — this keeps your infrastructure as code in sync with what's actually running.

**Why `kubectl rollout status` with a timeout?** This command blocks until the rollout completes (all new pods are ready). The `--timeout=10m` prevents the pipeline from hanging forever if the rollout gets stuck (e.g., image pull failure, readiness probe failing). A stuck rollout without a timeout would occupy the Jenkins agent for hours.

**Why a smoke test after deployment?** Even though Kubernetes checks liveness/readiness probes, these don't catch application-level bugs — a pod can be "healthy" from Kubernetes' perspective but return wrong answers. The `curl -f http://$URL/health` test validates the application's actual behavior.

**Why automatic rollback in `post { failure }`?** If the deployment succeeds but the smoke test fails, the `post { failure }` block triggers `kubectl rollout undo`, which reverts to the previous deployment state. This creates a safety net: if something is wrong with the new version, users are never exposed to it for more than a few seconds.

**Common pitfalls:**
- **Database schema changes:** If the new code expects a different DB schema, old pods (still running during rolling update) may crash. Solution: make schema changes backward-compatible (expand-contract pattern)
- **Session affinity:** Users might be bounced between old and new pod versions during the rollout, causing inconsistent behavior. Solution: use sticky sessions or maintain feature parity between versions
- **Long-running requests:** If a user request takes 5 minutes, terminating pods mid-request drops it. Solution: use preStop hooks to gracefully drain connections before pod termination
- **Resource constraints:** If the cluster doesn't have enough capacity to run both old and new pods simultaneously, the rollout may fail. Ensure cluster has enough headroom (e.g., 25% extra capacity)

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

### Detailed Explanation

Running infrastructure-as-code (IaC) through Jenkins provides accountability, auditability, and automation for infrastructure changes. Here's why this design is effective:

**Why the plan/apply split?** This is the most critical pattern for IaC pipelines. The `plan` step shows exactly what will change (resources to create, modify, or destroy) without actually making changes. A human reviews the plan output and must explicitly approve (`apply`) before any infrastructure changes happen. This prevents accidental destruction of production resources — a common risk when running Terraform directly from a laptop.

**Why `terraform init -backend-config=backend-prod.hcl`?** The `-backend-config` flag allows you to specify environment-specific backend configurations (S3 bucket, DynamoDB table for state locking) without hardcoding them in the Terraform code. This means the same Terraform module can be used for dev, staging, and prod — only the backend config file changes per environment. The backend file would typically reference environment-specific S3 buckets (e.g., `mycompany-terraform-state-prod`).

**Why stash the plan file?** The `terraform plan -out=tfplan` output is a binary file that contains the exact execution plan. By stashing it, you ensure the `apply` stage uses the exact same plan that was reviewed — no changes can sneak in between plan and apply. If someone commits a change between the plan and apply, the plan file would be rejected by Terraform (plan files are tied to specific state versions).

**Why restrict apply to `branch 'main'`?** Infrastructure changes are high-risk — they affect all users of the system. Requiring that applies only happen from `main` (after merge and code review) adds an extra layer of safety. Feature branch changes should only run `plan` (to preview changes), never `apply`.

**Why the `choice` parameter for ACTION?** The `choice` parameter gives operators a dropdown: `plan` or `apply`. This is intentional — you never want an operator to accidentally type "apply" when they meant "plan" (which could happen with a free-text string parameter). The dropdown makes the intention explicit.

**Production hardening:**
- **State locking:** Ensure DynamoDB is configured for state locking to prevent concurrent applies. The pipeline should fail if someone else is already running terraform apply
- **Plan comment on PR:** Use the `terraform plan` output to automatically comment on pull requests showing what infrastructure would change — this creates a reviewable artifact in the PR itself
- **Terraform version pinning:** Specify the exact Terraform version in the pipeline (using a tool like `tfenv`) to ensure consistency between CI and local runs
- **Workspace management:** Add a parameter for selecting the Terraform workspace (dev/staging/prod) — each workspace has its own state file
- **Cost estimation:** Integrate with tools like Infracost to show the cost impact of infrastructure changes before approval

### Detailed Explanation

Emergency hotfix pipelines are a delicate balance between speed and safety. Too many gates, and the hotfix takes too long; too few, and you risk deploying untested code. Here's the reasoning behind this design:

**Why validate the hotfix branch name?** Requiring the branch to start with `hotfix/` creates an explicit naming convention that distinguishes hotfixes from regular releases. This allows:
- Automated rules (e.g., "hotfix branches can skip security scan" instead of "all branches can skip")
- Clear audit trails — anyone looking at the deployment history can immediately identify hotfix deployments
- Monitoring dashboards can automatically flag hotfix deployments for review

**Why require a JIRA ticket in the commit message?** This is a guardrail that prevents "cowboy coding" — someone making an undocumented change directly to production without proper tracking. The `HOTFIX-\d+` pattern matches JIRA ticket IDs (e.g., `HOTFIX-1234`). This ensures:
- There's a documented reason for the hotfix
- Post-incident analysis can trace back to the originating issue
- Other teams are aware of the hotfix through the ticketing system

**Why skip integration tests but keep unit tests?** Integration tests are the longest-running part of a typical pipeline (often 30-60 minutes). Unit tests run in minutes. Skipping integration tests saves significant time, but keeping unit tests catches compilation errors and logic bugs that would cause immediate production issues. The `-DskipITs=true` Maven flag specifically skips integration tests while running unit tests with `mvn test` (not `mvn verify`, which runs both).

**Why skip staging?** Normal pipelines deploy to staging for validation before production. In a hotfix scenario, deploying to staging adds 10-15 minutes of deployment time plus validation time. By going directly to production (with the `--hotfix` flag), you cut that time significantly. The trade-off is accepting that the hotfix hasn't been validated in a staging environment — which is acceptable for a critical production bug where every minute of downtime costs money.

**Why the override parameter pattern?** The `SKIP_LONG_TESTS` parameter defaults to `true` but can be set to `false` if the engineer wants full testing. This gives flexibility — if the bug is in a trivial CSS change, skipping tests is safe; if it's in a core business logic change, the engineer can override to run full tests. The decision is deliberately left to human judgment rather than mandated by the pipeline.

**Why Slack notification in `post { always }`?** Hotfixes are exceptional events — the entire team needs to know immediately. The red Slack notification (`color: '#FF0000'`) with the hotfix branch name signals "something unusual just happened — pay attention." This also creates a communication channel for the team to discuss the hotfix post-deployment.

**Production enhancements:**
- **Time-bomb approval:** Add an automatic rollback after 24 hours — the hotfix deployment expires unless explicitly promoted to a permanent fix
- **Post-hotfix pipeline:** Automatically create a JIRA ticket to track the permanent fix, linking it to the hotfix ticket for traceability
- **Canary for hotfixes:** Instead of direct production deployment, use a canary strategy — deploy to 10% of instances first, verify, then roll out
- **Feature flag wrapper:** If the hotfix involves toggling behavior, wrap it in a feature flag so you can disable it without redeploying
- **Compliance recording:** Even though the hotfix bypasses normal gates, record the reason and authorization in a compliance log for audit purposes

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

### Detailed Explanation

In regulated industries (finance, healthcare, government), every deployment must be traceable and attributable. This pipeline is designed to produce evidence that satisfies auditors. Here's the reasoning:

**Why GPG-signed commits?** Auditors need to verify that the code deployed to production came from an authorized developer, not from an attacker who gained write access to the repository. A GPG-signed commit proves the commit was created by someone holding the private key associated with a verified identity (e.g., a company-issued GPG key stored on a hardware token). The `%G?` format returns `G` for a valid signature from a trusted key, `B` for a bad signature, `U` for a good signature from an untrusted key, and `N` for no signature.

**Why capture BUILD_USER and BUILD_USER_ID?** The `BUILD_USER` variable comes from the Jenkins Build User Vars plugin and captures who actually triggered the build (even if triggered via webhook). This is critical for audits — you need to know if the build was triggered by a developer clicking "Build Now", a webhook from a PR merge, or a cron schedule. This data is needed for SOC2, ISO 27001, and SOX compliance.

**Why the `submitterParameter` in the input step?** The `submitterParameter` captures WHO approved the deployment. In the audit report, this shows "Approved by: jane.doe" — not just "approved." This creates a clear chain of accountability. During an audit, the auditor can say "Jane approved deployment #1234" and Jane can confirm the decision.

**Why restrict approval to specific roles?** `submitter "compliance-officer,release-manager"` limits who can approve production deployments to authorized individuals. This prevents a well-meaning developer from accidentally approving their own deployment. In the event of a security incident, only authorized personnel could have approved the deployment — narrowing the investigation.

**What the audit report provides:**
- **Temporal evidence:** `Date` — proves when the deployment happened (checks: was it during normal business hours?)
- **Identity evidence:** `Triggered by` and `Approved by` — proves who initiated and authorized (checks: did two different people approve?)
- **Content integrity:** `Commit` — links the deployment to a specific commit hash (checks: can we reproduce what was deployed?)
- **Change traceability:** `Branch` — shows which branch was deployed (checks: was it a release branch or someone's feature branch?)
- **Outcome evidence:** `Build result` — captures whether the deployment succeeded or failed

**Production enhancements for deeper compliance:**
- **Immutable audit logs:** Write audit reports to an append-only store (AWS CloudWatch Logs, Splunk, or a blockchain-based logging service) so they can't be tampered with after the fact
- **Sign the audit report:** GPG-sign the audit report within the pipeline using Jenkins' GPG key, so auditors can verify the report was generated by Jenkins, not fabricated later
- **External evidence storage:** In addition to archiving in Jenkins, push the audit report to a separate storage (S3 with versioning enabled) that Jenkins admins can't delete — this prevents a rogue admin from covering their tracks
- **Periodic compliance validation:** Add a scheduled pipeline that scans historical builds for compliance gaps — for example, find builds that went to production without a signed commit or an approval
- **Real-time compliance dashboards:** Use Jenkins' Prometheus plugin to expose audit metrics (number of compliant vs. non-compliant deployments, approval times, etc.) for compliance dashboards

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
