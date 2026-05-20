---
title: "📝 Practice Test 3: SCM, Build Triggers & Credentials"
weight: 20
bookToc: false
---

# Practice Test 3: SCM, Build Triggers & Credentials

> **Simulate the real Jenkins certification exam with 57 questions. Time yourself — you have 60 minutes. Aim for 70% or higher.**

---

## 📋 Exam Instructions

| Detail | Value |
|--------|-------|
| **Questions** | 57 |
| **Time Limit** | 60 minutes |
| **Passing Score** | ~70% (40/57 correct) |
| **Format** | Multiple choice |

---
<div class="exam-controls">
  <div class="exam-timer" data-minutes="60" data-test-id="exam-test-3">
    <div class="timer-face">
      <span class="timer-minutes">60</span>
      <span class="timer-separator">:</span>
      <span class="timer-seconds">00</span>
    </div>
    <div class="timer-controls">
      <button class="timer-btn timer-start">▶ Start</button>
      <button class="timer-btn timer-pause">⏸ Pause</button>
      <button class="timer-btn timer-reset">↺ Reset</button>
    </div>
    <span class="timer-status">Time Remaining</span>
  </div>
  <div class="exam-progress-check">
    <input type="checkbox" class="test-complete-check" id="test-complete-3" data-test-id="exam-test-3">
    <label for="test-complete-3">✓ Mark test as completed</label>
  </div>
</div>

**57 Questions | Duration: 60 minutes | Focus: Source code management, build triggers, credentials management**

---

## Questions

**1. Which SCM systems does Jenkins natively support?**
- [ ] Git, SVN, Mercurial
- [ ] Git only
- [ ] Git, SVN, CVS
- [ ] SVN only

<details>
<summary>Answer</summary>
**Git, SVN, CVS**

Jenkins natively supports Git, Subversion (SVN), and CVS through built-in plugins. Additional SCM plugins are available for Mercurial, Perforce, TFS, and others.
</details>

---

**2. What is SCM polling in Jenkins?**
- [ ] A system where Jenkins checks the SCM at regular intervals for changes
- [ ] A voting system for code changes
- [ ] A manual code review process
- [ ] A deployment approval mechanism

<details>
<summary>Answer</summary>
**A system where Jenkins checks the SCM at regular intervals for changes**

SCM polling is a trigger mechanism where Jenkins periodically checks the source code repository for changes. If new commits are detected, Jenkins triggers a build. This is less efficient than webhooks.
</details>

---

**3. What is the H in a Jenkins cron expression `H */4 * * *`?**
- [ ] Hourly trigger
- [ ] Hash symbol for distributed scheduling
- [ ] High priority
- [ ] Half-hour

<details>
<summary>Answer</summary>
**Hash symbol for distributed scheduling**

The `H` (hash) symbol in Jenkins cron expressions distributes job execution across the specified time period. Instead of all jobs starting at exactly the same minute, each job's start time is hashed based on its name, reducing load spikes.
</details>

---

**4. What is the purpose of a GitHub webhook in Jenkins?**
- [ ] To send Jenkins build status to GitHub
- [ ] To trigger Jenkins builds when GitHub events (like pushes) occur
- [ ] To synchronize Jenkins and GitHub users
- [ ] To backup Jenkins to GitHub

<details>
<summary>Answer</summary>
**To trigger Jenkins builds when GitHub events (like pushes) occur**

GitHub webhooks send HTTP POST requests to Jenkins when configured events occur (push, pull request, release). Jenkins receives the webhook and triggers builds automatically. This is more efficient than SCM polling as builds start immediately.
</details>

---

**5. What are the main credential types in Jenkins?**
- [ ] Username/password, SSH keys, secret text, certificate
- [ ] Only passwords
- [ ] Only SSH keys
- [ ] Only API tokens

<details>
<summary>Answer</summary>
**Username/password, SSH keys, secret text, certificate**

Jenkins supports multiple credential types: Username with password, SSH keys (private key), Secret text (API tokens), Secret file (X.509 certificates), and more. These are stored encrypted in the Jenkins credential store.
</details>

---

**6. How does Git polling differ from webhook triggers?**
- [ ] Polling is real-time; webhooks have delays
- [ ] Polling checks on a schedule; webhooks push events immediately
- [ ] Polling is more efficient
- [ ] Webhooks require more configuration

<details>
<summary>Answer</summary>
**Polling checks on a schedule; webhooks push events immediately**

SCM polling periodically checks for changes (e.g., every 5 minutes), introducing delay between commit and build. Webhooks notify Jenkins immediately when changes happen, enabling near-instant build triggers.
</details>

---

**7. What is a Jenkins credential provider?**
- [ ] A service that generates credentials
- [ ] A storage backend for Jenkins credentials
- [ ] A credential validation service
- [ ] A password manager

<details>
<summary>Answer</summary>
**A storage backend for Jenkins credentials**

Credential providers are storage backends for Jenkins credentials. Built-in providers include the Jenkins internal store. Additional providers like HashiCorp Vault, CyberArk, or Azure Key Vault can be integrated via plugins.
</details>

---

**8. What is the purpose of the `scm` step in a Pipeline?**
- [ ] To test SCM connectivity
- [ ] To configure the SCM settings for the pipeline
- [ ] To clone the repository
- [ ] To display SCM information

<details>
<summary>Answer</summary>
**To configure the SCM settings for the pipeline**

The `scm` step is used to define the source code management configuration for a pipeline. It specifies the repository URL, credentials, branch, and other SCM-specific settings. The `checkout` step uses the SCM configuration to clone the repository.
</details>

---

**9. How do you configure Jenkins to use a specific Git branch?**
- [ ] In the job configuration under Source Code Management
- [ ] By setting a parameter in the build script
- [ ] By editing the Jenkins config.xml
- [ ] By using a plugin

<details>
<summary>Answer</summary>
**In the job configuration under Source Code Management**

For freestyle jobs and pipelines, the branch to build is specified in the SCM configuration section. You can use wildcards like `*/main` or `*/develop`, or reference a parameter like `$BRANCH_NAME`.
</details>

---

**10. What is SCM behavior in multibranch pipelines?**
- [ ] All branches are built with the same Jenkinsfile
- [ ] Each branch can have its own Jenkinsfile
- [ ] Only the main branch is built
- [ ] Branches cannot be built automatically

<details>
<summary>Answer</summary>
**All branches are built with the same Jenkinsfile**

In multibranch pipelines, Jenkins automatically discovers branches and creates a pipeline for each branch that contains a Jenkinsfile. The Jenkinsfile can contain branch-specific logic using `when { branch }` conditions.
</details>

---

**11. What does the `git` step do in a Pipeline?**
- [ ] Configures Git settings
- [ ] Clones a Git repository
- [ ] Installs Git on the agent
- [ ] Updates Git to the latest version

<details>
<summary>Answer</summary>
**Clones a Git repository**

The `git` step is a simplified way to clone a Git repository. It accepts parameters like `url`, `branch`, `credentialsId`, and `changelog`. It is a wrapper around the more verbose `checkout` step with GitSCM.
</details>

---

**12. What is the purpose of Git credentials binding?**
- [ ] To encrypt Git operations
- [ ] To securely authenticate Jenkins with Git repositories
- [ ] To create Git users
- [ ] To manage Git SSH keys

<details>
<summary>Answer</summary>
**To securely authenticate Jenkins with Git repositories**

Git credentials binding allows Jenkins to securely authenticate with Git repositories using stored credentials (username/password, SSH keys, or access tokens). This avoids hardcoding credentials in Jenkinsfiles.
</details>

---

**13. What is a webhook payload in Jenkins?**
- [ ] The response from Jenkins to a webhook
- [ ] The HTTP request data sent by the SCM to Jenkins
- [ ] A hook for web developers
- [ ] A Jenkins plugin

<details>
<summary>Answer</summary>
**The HTTP request data sent by the SCM to Jenkins**

A webhook payload is the HTTP POST request sent by GitHub, GitLab, or Bitbucket to Jenkins when a configured event occurs. The payload contains information about the event, including commit details, branch, and repository.
</details>

---

**14. How do you verify Jenkins is receiving webhook requests?**
- [ ] By checking the Jenkins system log
- [ ] By monitoring the `/github-webhook/` endpoint logs
- [ ] By reviewing the build history
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Webhook activity can be verified by checking Jenkins system logs, monitoring the webhook endpoint logs (e.g., `/github-webhook/`), and reviewing webhook delivery logs on the SCM provider's side (GitHub/GitLab).
</details>

---

**15. What is the purpose of the `checkout` step in a Pipeline?**
- [ ] To validate the SCM configuration
- [ ] To clone the repository and optionally switch branches
- [ ] To check for updates
- [ ] To verify credentials

<details>
<summary>Answer</summary>
**To clone the repository and optionally switch branches**

The `checkout` step is the most flexible way to clone a repository. It accepts an SCM configuration object that defines all checkout parameters including URL, credentials, branch, changelog, and submodule settings.
</details>

---

**16. What is sparse checkout in Git?**
- [ ] Cloning only specific files or directories instead of the entire repository
- [ ] Cloning the repository with limited history
- [ ] A lightweight Git installation
- [ ] A partial branch checkout

<details>
<summary>Answer</summary>
**Cloning only specific files or directories instead of the entire repository**

Sparse checkout allows cloning only specific paths from a Git repository. This reduces the clone time and disk space usage, particularly useful for large monorepos where only a subset of files is needed for the build.
</details>

---

**17. How do you configure Jenkins to use shallow Git clones?**
- [ ] Using the `shallow()` option in GitSCM
- [ ] By setting the `depth` parameter in the Git checkout
- [ ] Using a plugin
- [ ] By editing the system configuration

<details>
<summary>Answer</summary>
**By setting the `depth` parameter in the Git checkout**

Shallow clones can be configured in Jenkins by setting the `depth` parameter in the Git checkout configuration. This limits the number of commits fetched, reducing clone time and disk usage for repositories with long histories.
</details>

---

**18. What is the `Polling ignores commits in certain paths` feature?**
- [ ] An option to skip polling for changes in specific file paths
- [ ] A feature to ignore all commit polling
- [ ] A way to skip certain repositories
- [ ] A method to filter webhook notifications

<details>
<summary>Answer</summary>
**An option to skip polling for changes in specific file paths**

This feature in the Git plugin allows specifying excluded paths. Changes to files in those paths (like documentation, README, or configuration files) will not trigger a build, preventing unnecessary builds for non-code changes.
</details>

---

**19. What is the purpose of the "Merge before build" option?**
- [ ] To run builds in parallel
- [ ] To merge the feature branch with the target branch before building
- [ ] To merge build artifacts
- [ ] To combine multiple Jenkins jobs

<details>
<summary>Answer</summary>
**To merge the feature branch with the target branch before building**

The "Merge before build" option (in the Git plugin) creates a temporary merge of the source branch with the target branch and builds the merged result. This validates that the code works correctly when merged, catching integration issues early.
</details>

---

**20. What is the advantage of using SSH keys for Git authentication over passwords?**
- [ ] SSH keys are faster
- [ ] SSH keys provide stronger security and can be used without interactive login
- [ ] SSH keys are easier to manage
- [ ] SSH keys work with all Git providers

<details>
<summary>Answer</summary>
**SSH keys provide stronger security and can be used without interactive login**

SSH keys offer stronger security through public-key cryptography and enable non-interactive Git operations. They are ideal for automated systems like Jenkins where password-based authentication would require storing passwords insecurely.
</details>

---

**21. What is a Personal Access Token (PAT) in Git context?**
- [ ] A temporary password for Git operations
- [ ] An alternative to passwords for Git authentication with scoped permissions
- [ ] A token for Git GUI clients
- [ ] A session cookie

<details>
<summary>Answer</summary>
**An alternative to passwords for Git authentication with scoped permissions**

A Personal Access Token is a token-based authentication method that allows restricting permissions (read, write, admin) and can be revoked independently of the user account. GitHub, GitLab, and Bitbucket all support PATs.
</details>

---

**22. What is the difference between `checkout` and `git` steps?**
- [ ] `checkout` is simpler; `git` is more configurable
- [ ] `git` is simpler; `checkout` provides more options
- [ ] They are identical
- [ ] `git` is for scripted pipelines; `checkout` is for declarative

<details>
<summary>Answer</summary>
**`git` is simpler; `checkout` provides more options**

The `git` step is a simplified wrapper for common Git checkout operations. The `checkout` step with GitSCM provides full control over checkout options including submodules, shallow clones, changelog, and advanced SCM settings.
</details>

---

**23. What is the purpose of the `changelog` option in Git checkout?**
- [ ] To generate a changelog file
- [ ] To retrieve commit changes since the last build
- [ ] To log checkout operations
- [ ] To display Git version

<details>
<summary>Answer</summary>
**To retrieve commit changes since the last build**

The `changelog` option controls whether Jenkins retrieves and displays commit information (changelog) since the last build. Enabling this provides detailed change information in build pages.
</details>

---

**24. What is Git LFS and how does Jenkins support it?**
- [ ] Git Large File Storage; supported via the Git LFS plugin
- [ ] Git Local File System; not supported
- [ ] Git Lightweight File System; built-in
- [ ] Git Logging File System; requires configuration

<details>
<summary>Answer</summary>
**Git Large File Storage; supported via the Git LFS plugin**

Git LFS (Large File Storage) replaces large files (binaries, archives, media) with text pointers while storing the actual file content on a remote server. Jenkins supports LFS through the Git LFS plugin.
</details>

---

**25. What does the `clean before checkout` option do?**
- [ ] Cleans the Jenkins workspace before cloning
- [ ] Cleans the Git repository
- [ ] Deletes old builds
- [ ] Removes stale credentials

<details>
<summary>Answer</summary>
**Cleans the Jenkins workspace before cloning**

The `clean before checkout` option forces a fresh checkout by deleting all untracked files and reverting changes in the workspace before cloning. This ensures builds start from a clean state, preventing contamination from previous builds.
</details>

---

**26. How do you trigger a build on a pull request in Jenkins?**
- [ ] Using a webhook for pull request events
- [ ] Using SCM polling
- [ ] Using a scheduled trigger
- [ ] Using a manual trigger

<details>
<summary>Answer</summary>
**Using a webhook for pull request events**

Pull request builds are typically triggered via webhooks configured to fire on pull request events (opened, updated, merged). Multibranch Pipeline and GitHub Organization Folder projects automatically build pull requests when configured.
</details>

---

**27. What is the purpose of the GitHub Checks API?**
- [ ] To validate GitHub credentials
- [ ] To report build status as checks in GitHub pull requests
- [ ] To check for GitHub API limits
- [ ] To verify GitHub webhooks

<details>
<summary>Answer</summary>
**To report build status as checks in GitHub pull requests**

The GitHub Checks API allows Jenkins to report detailed build status directly in GitHub pull requests. This shows individual check results (build, test, lint), logs, and annotations inline in the GitHub UI.
</details>

---

**28. What is the role of `credentialsId` in the `git` step?**
- [ ] To specify the credential ID stored in Jenkins for authentication
- [ ] To generate credentials
- [ ] To validate credentials
- [ ] To delete credentials

<details>
<summary>Answer</summary>
**To specify the credential ID stored in Jenkins for authentication**

The `credentialsId` parameter in the `git` step references a stored credential in Jenkins by its ID. Jenkins looks up the credential and uses it for authentication when cloning the repository.
</details>

---

**29. What is Git submodule support in Jenkins?**
- [ ] A feature to recursively clone Git submodules
- [ ] A feature to create Git submodules
- [ ] A feature to convert repositories to submodules
- [ ] A feature to delete submodules

<details>
<summary>Answer</summary>
**A feature to recursively clone Git submodules**

Git submodule support in Jenkins recursively checks out submodules after the main repository is cloned. The Git plugin provides options to control which submodules are checked out and how they are updated.
</details>

---

**30. How do you prevent Jenkins from building on documentation-only changes?**
- [ ] Using the "Polling ignores commits in certain paths" option
- [ ] Using a `when` condition with `changeset`
- [ ] Both A and B
- [ ] This is not possible

<details>
<summary>Answer</summary>
**Both A and B**

Jenkins can skip builds for documentation changes using either the Git plugin's excluded paths option (at checkout time) or the `when { changeset }` directive in Declarative Pipeline (at stage execution time).
</details>

---

**31. What is the purpose of the `pollSCM` trigger in a Pipeline?**
- [ ] To send SCM polling results to a monitoring system
- [ ] To enable periodic SCM polling for build triggers
- [ ] To check SCM server health
- [ ] To poll for new plugins

<details>
<summary>Answer</summary>
**To enable periodic SCM polling for build triggers**

The `pollSCM` trigger within the `triggers` directive enables Jenkins to periodically poll the SCM for changes. If changes are detected, the pipeline is automatically triggered.
</details>

---

**32. What is the difference between Git polling and a cron trigger?**
- [ ] Git polling checks for SCM changes; cron triggers are time-based regardless of changes
- [ ] They are identical
- [ ] Git polling is more reliable
- [ ] Cron triggers are only for freestyle jobs

<details>
<summary>Answer</summary>
**Git polling checks for SCM changes; cron triggers are time-based regardless of changes**

Git polling triggers builds only when changes are detected in the repository. Cron triggers run builds on a fixed schedule (e.g., nightly) regardless of whether code has changed.
</details>

---

**33. What is a build trigger chain in Jenkins?**
- [ ] A series of builds triggered sequentially based on completion
- [ ] Multiple builds triggered concurrently
- [ ] A build with multiple stages
- [ ] A build that runs on multiple agents

<details>
<summary>Answer</summary>
**A series of builds triggered sequentially based on completion**

A build trigger chain is a sequence where one job triggers another upon completion. This creates dependencies forming a chain (A triggers B, B triggers C), which is useful for multi-stage delivery pipelines with freestyle jobs.
</details>

---

**34. How do you pass parameters from one job to another in a trigger chain?**
- [ ] Using the `build` step with parameters
- [ ] Using environment variables
- [ ] Using shared files
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Parameters can be passed between jobs using the `build` step with parameters, environment variables set in one job and read in another, or files written to shared storage. The `build` step with explicit parameters is the most reliable approach.
</details>

---

**35. What is a post-commit hook in Git?**
- [ ] A script that runs on the Git server after a commit
- [ ] A script that runs on the client after a commit
- [ ] A Jenkins plugin
- [ ] A Git command

<details>
<summary>Answer</summary>
**A script that runs on the client after a commit**

A post-commit hook is a client-side Git hook script that runs after a commit is created locally. It can be used to trigger Jenkins builds from the developer's machine, though server-side webhooks are more common for CI/CD.
</details>

---

**36. What is the purpose of the "Force polling using workspace" option in Jenkins Git plugin?**
- [ ] To use the agent workspace for polling instead of the controller
- [ ] To force a full workspace recheck
- [ ] To force a Git pull instead of fetch
- [ ] To force checkout of a specific branch

<details>
<summary>Answer</summary>
**To use the agent workspace for polling instead of the controller**

This option uses the agent's workspace (instead of the controller) to perform SCM polling. This can improve polling performance for large repositories by utilizing agent resources and local caching.
</details>

---

**37. What is the purpose of the credentials binding plugin?**
- [ ] To create credentials in Jenkins
- [ ] To securely inject credentials into build environments
- [ ] To delete expired credentials
- [ ] To audit credential usage

<details>
<summary>Answer</summary>
**To securely inject credentials into build environments**

The Credentials Binding plugin provides the `withCredentials` step for securely injecting stored credentials as environment variables. This allows pipelines to use secrets without exposing them in code or logs.
</details>

---

**38. What is the default Git branch in most modern Git hosting platforms?**
- [ ] master
- [ ] main
- [ ] develop
- [ ] production

<details>
<summary>Answer</summary>
**main**

GitHub, GitLab, and Bitbucket have migrated from `master` to `main` as the default branch name for new repositories, following community-wide efforts to adopt more inclusive terminology.
</details>

---

**39. How do you configure Jenkins to use a specific Git credential globally?**
- [ ] Under Jenkins > Manage Credentials > Global Credentials
- [ ] In each job configuration individually
- [ ] By editing Jenkins configuration files
- [ ] Using environment variables

<details>
<summary>Answer</summary>
**Under Jenkins > Manage Credentials > Global Credentials**

Global credentials are configured in Jenkins under Manage Jenkins > Manage Credentials > Global Credentials (unrestricted). These credentials can be used by any job or pipeline on the Jenkins instance.
</details>

---

**40. What is the role of the `SCM API` plugin in Jenkins?**
- [ ] To provide a unified API for different SCM systems
- [ ] To connect Jenkins to SCM databases
- [ ] To create SCM repositories
- [ ] To monitor SCM performance

<details>
<summary>Answer</summary>
**To provide a unified API for different SCM systems**

The SCM API plugin provides common abstractions for integrating various SCM systems with Jenkins. It defines interfaces that SCM plugins implement, enabling consistent behavior across Git, SVN, and other SCM providers.
</details>

---

**41. What is a Git merge conflict in Jenkins context?**
- [ ] An error when Jenkins updates plugins
- [ ] A conflict between different code changes that cannot be automatically resolved
- [ ] A Jenkins configuration conflict
- [ ] A network connectivity issue

<details>
<summary>Answer</summary>
**A conflict between different code changes that cannot be automatically resolved**

A merge conflict occurs when Git cannot automatically merge changes from different branches. In Jenkins, this may happen during "Merge before build" operations, causing the build to fail until the conflict is manually resolved.
</details>

---

**42. What does the `gitTool` option do in the Git checkout?**
- [ ] Specifies which Git installation to use
- [ ] Installs Git on the agent
- [ ] Configures Git settings
- [ ] Updates Git to a specific version

<details>
<summary>Answer</summary>
**Specifies which Git installation to use**

The `gitTool` option in the Git plugin allows specifying the path or named Git installation to use for the checkout operation. Multiple Git versions can be configured in Global Tool Configuration.
</details>

---

**43. What is GitWeb in Jenkins context?**
- [ ] A web-based Git repository viewer integrated with Jenkins
- [ ] A Jenkins plugin for Git webhooks
- [ ] A Git server
- [ ] A web-based Git client

<details>
<summary>Answer</summary>
**A web-based Git repository viewer integrated with Jenkins**

GitWeb is a simple web-based Git repository viewer that can be integrated with Jenkins to browse repository content and history directly from the Jenkins UI, though it is less commonly used now.
</details>

---

**44. How do you set up Jenkins to build only specific folder paths in a monorepo?**
- [ ] Using file path filters in SCM configuration
- [ ] Using multiple Jenkinsfiles per folder
- [ ] Using the `when { changeset }` directive
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Monorepo conditional builds can be achieved through file path filters in Git SCM configuration, multiple Jenkinsfiles with path-specific triggers, or the `when { changeset }` directive in Declarative Pipeline to skip stages that don't affect specific paths.
</details>

---

**45. What is the purpose of the `gitParameter` plugin?**
- [ ] To add Git-related build parameters to parameterized jobs
- [ ] To configure Git settings
- [ ] To manage Git credentials
- [ ] To optimize Git performance

<details>
<summary>Answer</summary>
**To add Git-related build parameters to parameterized jobs**

The Git Parameter plugin adds dynamic Git parameters to build jobs, such as branch selectors, tag selectors, and revision selectors. These are populated dynamically from the Git repository when the build is triggered.
</details>

---

**46. What is a Jenkins multibranch pipeline?**
- [ ] A pipeline that automatically discovers and builds branches
- [ ] A pipeline with multiple stages
- [ ] A pipeline that runs on multiple agents
- [ ] A pipeline with multiple SCM sources

<details>
<summary>Answer</summary>
**A pipeline that automatically discovers and builds branches**

A multibranch pipeline automatically discovers branches in a Git repository and creates a pipeline for each branch that contains a Jenkinsfile. It manages branch indexing, automatic builds, and cleanup for deleted branches.
</details>

---

**47. What is the purpose of branch indexing in multibranch pipelines?**
- [ ] To index the Jenkinsfile for syntax highlighting
- [ ] To scan the repository for branches and create/delete pipelines accordingly
- [ ] To index build artifacts
- [ ] To measure branch performance

<details>
<summary>Answer</summary>
**To scan the repository for branches and create/delete pipelines accordingly**

Branch indexing scans the Git repository to discover new branches, create corresponding pipelines, and delete pipelines for removed branches. It runs periodically or can be triggered via webhooks.
</details>

---

**48. How do you configure Jenkins to build only specific branches in a multibranch pipeline?**
- [ ] Using branch filters (e.g., `main`, `develop`, `feature/*`)
- [ ] By manually creating jobs for each branch
- [ ] By editing Jenkins configuration
- [ ] By using environment variables

<details>
<summary>Answer</summary>
**Using branch filters (e.g., `main`, `develop`, `feature/*`)**

Multibranch pipeline projects support branch filters that specify which branches should be built. Filters can use branch name patterns, regular expressions, or whitelist/blacklist strategies.
</details>

---

**49. What is the purpose of the `discoverBranches` strategy in multibranch pipelines?**
- [ ] To discover branches from the SCM
- [ ] To choose which branch discovery strategy to use (e.g., all branches, only main)
- [ ] To rename branches
- [ ] To monitor branch growth

<details>
<summary>Answer</summary>
**To choose which branch discovery strategy to use (e.g., all branches, only main)**

The `discoverBranches` strategy defines how Jenkins discovers branches. Options include discovering all branches, only main/master branches, or branches matching specific naming patterns.
</details>

---

**50. What is a GitHub Organization Folder in Jenkins?**
- [ ] A folder that organizes GitHub repositories in Jenkins
- [ ] A project type that discovers all repositories in a GitHub org and creates pipelines
- [ ] A folder for storing GitHub configuration
- [ ] A GitHub repository organization tool

<details>
<summary>Answer</summary>
**A project type that discovers all repositories in a GitHub org and creates pipelines**

A GitHub Organization Folder automatically scans a GitHub organization or account, discovers all repositories containing Jenkinsfiles, and creates multibranch pipelines for each. It fully automates project creation for GitHub-hosted code.
</details>

---

**51. What does the SCM step `checkout scm: scmGit(...)` do?**
- [ ] Checks out code using Git with explicit configuration
- [ ] Checks out the default SCM configured in the job
- [ ] Checks out code from SVN
- [ ] Validates SCM configuration

<details>
<summary>Answer</summary>
**Checks out code using Git with explicit configuration**

`checkout scm: scmGit(...)` uses the Git pipeline step to explicitly configure and perform a Git checkout. This provides full control over checkout options including repository URL, credentials, branch, and clone behavior.
</details>

---

**52. What is the purpose of the `generic-webhook-trigger` plugin?**
- [ ] To trigger Jenkins jobs from any webhook source
- [ ] To send webhook notifications
- [ ] To generate webhook URLs
- [ ] To test webhook connectivity

<details>
<summary>Answer</summary>
**To trigger Jenkins jobs from any webhook source**

The Generic Webhook Trigger plugin allows triggering Jenkins jobs from any system that can send HTTP POST requests. It supports extracting parameters from JSON/XML payloads and filtering based on payload content.
</details>

---

**53. How do you filter webhook triggers based on payload content?**
- [ ] By using the optional filter expressions in the webhook trigger configuration
- [ ] By using regular expressions on the request body
- [ ] By using `when` conditions in the pipeline
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Webhook triggers can be filtered at multiple levels: the generic webhook trigger plugin supports regex filters on payload content, pipelines can use `when` conditions to check variables, and some SCM webhooks support event-type filtering.
</details>

---

**54. What is the function of the `secret text` credential type?**
- [ ] To store passwords
- [ ] To store API tokens and other generic secrets
- [ ] To store SSH keys
- [ ] To store certificates

<details>
<summary>Answer</summary>
**To store API tokens and other generic secrets**

The "Secret text" credential type stores arbitrary text secrets like API tokens, access keys, or any sensitive string. These are encrypted at rest in Jenkins and masked in logs and UI.
</details>

---

**55. What is the purpose of the "Secret file" credential type in Jenkins?**
- [ ] To upload and store a file containing sensitive data
- [ ] To store secrets in a file format
- [ ] To encrypt build files
- [ ] To store binary secrets

<details>
<summary>Answer</summary>
**To upload and store a file containing sensitive data**

The "Secret file" credential type allows uploading an entire file (like a service account JSON key, certificate file, or SSH private key) as a credential. Jenkins securely stores the file and makes it available to builds.
</details>

---

**56. How do you revoke credentials in Jenkins?**
- [ ] By deleting the credential from the credentials store
- [ ] By disabling the credential
- [ ] By setting an expiry date
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Credentials can be revoked in Jenkins by deleting them from the credential store, disabling them, or setting an expiration date. It's also recommended to rotate or revoke the underlying secret (e.g., GitHub token) independently.
</details>

---

**57. What is best practice for credential rotation in Jenkins?**
- [ ] Rotate credentials periodically and update them in Jenkins
- [ ] Never change credentials once set
- [ ] Use the same credential for all systems
- [ ] Store credentials in Jenkinsfiles

<details>
<summary>Answer</summary>
**Rotate credentials periodically and update them in Jenkins**

Best practice is to rotate credentials on a regular schedule (e.g., every 90 days) and update them in the Jenkins credential store. Automation tools or plugins can help manage credential rotation, and external providers like HashiCorp Vault support dynamic secrets.
</details>

---

> **Pro tip:** After completing this test, check your score per topic. If you scored below 70% in any area, review the corresponding chapters in this guide before attempting the next practice test.
