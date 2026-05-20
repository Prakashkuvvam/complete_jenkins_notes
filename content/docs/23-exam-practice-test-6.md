---
title: "📝 Practice Test 6: Production Jenkins & Monitoring"
weight: 23
bookToc: false
---

# Practice Test 6: Production Jenkins & Monitoring

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
  <div class="exam-timer" data-minutes="60" data-test-id="exam-test-6">
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
    <input type="checkbox" class="test-complete-check" id="test-complete-6" data-test-id="exam-test-6">
    <label for="test-complete-6">✓ Mark test as completed</label>
  </div>
</div>

**57 Questions | Duration: 60 minutes | Focus: Production-grade Jenkins, monitoring, high availability, backup, pipeline optimization**

---

## Questions

**1. What is the recommended Jenkins backup strategy?**
- [ ] Backup only job configurations
- [ ] Backup JENKINS_HOME directory regularly
- [ ] Backup only plugins
- [ ] Backup is not necessary

<details>
<summary>Answer</summary>
**Backup JENKINS_HOME directory regularly**

The complete JENKINS_HOME directory should be backed up regularly, including job configurations, build logs, plugins, credentials, and system configuration. Automated scripts or backup plugins can handle this process.
</details>

---

**2. What is the most critical component to backup in JENKINS_HOME?**
- [ ] plugins/
- [ ] jobs/
- [ ] secrets/ (master.key and hudson.util.Secret)
- [ ] workspace/

<details>
<summary>Answer</summary>
**secrets/ (master.key and hudson.util.Secret)**

The `secrets/` directory containing `master.key` and `hudson.util.Secret` is critical because these files encrypt all stored credentials. Without them, credentials cannot be decrypted and are lost forever.
</details>

---

**3. What is the purpose of the ThinBackup plugin?**
- [ ] To perform automated, configurable backups of JENKINS_HOME
- [ ] To compress build artifacts
- [ ] To reduce Jenkins memory usage
- [ ] To clean up old builds

<details>
<summary>Answer</summary>
**To perform automated, configurable backups of JENKINS_HOME**

The ThinBackup plugin automates Jenkins backups, supporting scheduled backups, backup rotation, and selective backup of specific directories. It can back up to local or network storage locations.
</details>

---

**4. How do you monitor Jenkins health?**
- [ ] Using the Jenkins Monitoring plugin
- [ ] Using external monitoring tools (Prometheus, Grafana)
- [ ] Using Jenkins built-in health check URL
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Jenkins health can be monitored through the Monitoring plugin (resource usage dashboard), external tools like Prometheus/Grafana (using the Prometheus plugin), and the built-in health check URL (`/oops` or `/login` endpoint).
</details>

---

**5. What is a Jenkins health check URL?**
- [ ] `/health` — returns Jenkins server health status
- [ ] `/oops` or `/<any-url>` — Jenkins returns error pages for unhealthy states
- [ ] `/monitoring` — monitoring dashboard
- [ ] `/status` — server status page

<details>
<summary>Answer</summary>
**`/oops` or `/<any-url>` — Jenkins returns error pages for unhealthy states**

Jenkins doesn't have a dedicated health endpoint by default. A common health check approach is to check any URL — a healthy Jenkins returns 200 OK on valid pages, while an unhealthy one may return error pages. The `/oops` endpoint shows error details.
</details>

---

**6. What does the Jenkins Monitoring plugin track?**
- [ ] CPU, memory, and disk usage of the Jenkins controller
- [ ] Build queue length and build durations
- [ ] System load averages
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

The Monitoring plugin tracks system resources (CPU, memory, swap, disk), Jenkins-specific metrics (build queue size, executor counts, job durations), and provides historical graphs and alerts.
</details>

---

**7. What is the Jenkins Prometheus plugin?**
- [ ] A plugin that exposes Jenkins metrics in Prometheus format
- [ ] A plugin for creating Prometheus dashboards
- [ ] A plugin for managing Prometheus alerts
- [ ] A plugin for Prometheus job scheduling

<details>
<summary>Answer</summary>
**A plugin that exposes Jenkins metrics in Prometheus format**

The Prometheus plugin exposes Jenkins metrics (build counts, queue sizes, executor counts, job durations) in Prometheus-compatible format, allowing integration with standard monitoring stacks like Grafana.
</details>

---

**8. What is a common Jenkins performance issue?**
- [ ] Too many jobs consuming controller resources
- [ ] Long pipeline queue times due to insufficient executors
- [ ] Frequent full garbage collection in the JVM
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Common Jenkins performance issues include heavy resource usage from many concurrent jobs, build queue backlog from insufficient agent capacity, and JVM GC issues from memory pressure (especially with many plugins and jobs).
</details>

---

**9. How do you improve Jenkins pipeline performance?**
- [ ] Use parallel stages for independent tasks
- [ ] Optimize Docker image build caching
- [ ] Minimize agent workspace cleanups
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Pipeline performance can be improved by: running independent tasks in parallel, leveraging Docker layer caching for builds, reducing unnecessary workspace cleanup, using shallow Git clones, and caching dependencies between builds.
</details>

---

**10. What is the purpose of the Pipeline Graph Analysis plugin?**
- [ ] To visualize pipeline execution and identify bottlenecks
- [ ] To create pipeline diagrams
- [ ] To monitor pipeline resources
- [ ] To generate pipeline reports

<details>
<summary>Answer</summary>
**To visualize pipeline execution and identify bottlenecks**

The Pipeline Graph Analysis plugin provides visual representations of pipeline execution, showing stage durations, parallel branches, and overall pipeline flow. This helps identify performance bottlenecks and optimize pipeline structure.
</details>

---

**11. What is the recommended approach for handling large Jenkins instances?**
- [ ] Use a single large controller
- [ ] Implement vertical scaling (bigger server)
- [ ] Implement horizontal scaling and best practices
- [ ] Monitor and scale as needed

<details>
<summary>Answer</summary>
**Implement horizontal scaling and best practices**

For large instances, best practices include: using a controller with 0 executors, offloading work to agents, using Jenkins Operations Center or a team-managed controller model, implementing JCasC for reproducibility, and regular maintenance.
</details>

---

**12. What is the Jenkins build queue?**
- [ ] A queue of builds waiting for an available executor
- [ ] A queue of build artifacts
- [ ] A queue of Jenkins users
- [ ] A queue of plugin updates

<details>
<summary>Answer</summary>
**A queue of builds waiting for an available executor**

The build queue holds jobs that have been triggered but cannot run because no suitable agent/executor is available. Builds in the queue are displayed on the Jenkins dashboard with their position and estimated wait time.
</details>

---

**13. What causes a long build queue in Jenkins?**
- [ ] Insufficient agent capacity or configuration
- [ ] Too many concurrent build triggers
- [ ] Pipeline stages blocking executors
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Long build queues can result from insufficient agents or executors, a high rate of build triggers, long-running builds occupying executors, or misconfigured agent labels preventing job-to-agent matching.
</details>

---

**14. What is the purpose of the `disableConcurrentBuilds` option?**
- [ ] To prevent multiple builds of the same job from running simultaneously
- [ ] To disable all concurrent building
- [ ] To reduce server load
- [ ] To improve build stability

<details>
<summary>Answer</summary>
**To prevent multiple builds of the same job from running simultaneously**

`disableConcurrentBuilds` ensures that only one build of a job runs at a time. If a new build is triggered while another is running, it is queued until the running build completes. This is useful for deployments or resource-intensive jobs.
</details>

---

**15. What is build discarder configuration?**
- [ ] A setting to automatically delete old builds based on count or age
- [ ] A setting to discard build artifacts
- [ ] A setting to delete failed builds
- [ ] A setting to archive build history

<details>
<summary>Answer</summary>
**A setting to automatically delete old builds based on count or age**

The build discarder automatically removes old builds to manage disk space. It can be configured by maximum number of builds to keep, maximum age, or both. This prevents JENKINS_HOME from growing indefinitely.
</details>

---

**16. What is the purpose of the `logRotator` in Jenkins?**
- [ ] To rotate and compress build logs
- [ ] To delete old logs
- [ ] To archive logs to external storage
- [ ] All of the above

<details>
<summary>Answer</summary>
**To rotate and compress build logs**

The `logRotator` manages build log retention by specifying the number of builds to keep and the number of days to retain logs. It's configured in the job's post-build actions or in the pipeline's `options` directive.
</details>

---

**17. What is the best practice for managing JENKINS_HOME disk space?**
- [ ] Set up build discarders on all jobs
- [ ] Clean up old workspaces
- [ ] Archive artifacts to external storage
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Managing disk space requires a multi-pronged approach: configure build discarders on all jobs, implement workspace cleanup (especially on ephemeral agents), archive important artifacts externally (S3, Nexus, Artifactory), and monitor disk usage.
</details>

---

**18. What is the purpose of the `workspace-cleanup` plugin?**
- [ ] To delete workspace directories after builds
- [ ] To organize workspace directories
- [ ] To compress workspace files
- [ ] To back up workspace files

<details>
<summary>Answer</summary>
**To delete workspace directories after builds**

The Workspace Cleanup plugin provides the `cleanWs()` step to delete workspace directories. It runs in post-build actions or `post { always { cleanWs() } }` in pipelines, freeing disk space on agents.
</details>

---

**19. How do you handle Jenkins controller failover?**
- [ ] Using Jenkins high availability configurations
- [ ] By maintaining a warm standby controller
- [ ] Using shared storage for JENKINS_HOME
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Jenkins failover strategies include: maintaining a standby controller with synchronized JENKINS_HOME (using shared storage like NFS), using external databases for job data, and implementing automated failover processes with DNS/load balancer switching.
</details>

---

**20. Does Jenkins support true high availability?**
- [ ] No, Jenkins does not natively support multi-controller HA
- [ ] Yes, through clustering
- [ ] Yes, through replication
- [ ] Yes, through load balancing

<details>
<summary>Answer</summary>
**No, Jenkins does not natively support multi-controller HA**

Jenkins does not natively support active-active high availability. The recommended approach is to use active-passive with a warm standby, implement regular backups, and use CloudBees CI for enterprise HA requirements.
</details>

---

**21. What is the Jenkins Operations Center (CloudBees CI)?**
- [ ] A management layer for multiple Jenkins controllers
- [ ] A monitoring tool
- [ ] A job scheduler
- [ ] A backup tool

<details>
<summary>Answer</summary>
**A management layer for multiple Jenkins controllers**

CloudBees CI (formerly Jenkins Operations Center) provides a centralized management platform for multiple Jenkins controllers. It offers HA, disaster recovery, workload management, and governance across a fleet of controllers.
</details>

---

**22. What is the command to restart Jenkins?**
- [ ] `sudo systemctl restart jenkins`
- [ ] `http://jenkins-url/restart`
- [ ] `jenkins restart`
- [ ] Both A and B

<details>
<summary>Answer</summary>
**Both A and B**

Jenkins can be restarted via the OS service manager (`systemctl restart jenkins` on Linux, `net stop jenkins && net start jenkins` on Windows) or through the Jenkins UI at `/restart` (if the user has admin permissions).
</details>

---

**23. What is the purpose of Jenkins safe restart?**
- [ ] To restart after all running builds complete
- [ ] To restart immediately regardless of running builds
- [ ] To restart only the UI
- [ ] To restart only plugins

<details>
<summary>Answer</summary>
**To restart after all running builds complete**

A safe restart (`/safeRestart`) waits for all currently running builds to complete before restarting Jenkins. This ensures ongoing builds are not interrupted. Running builds must finish or be manually stopped.
</details>

---

**24. What is the Jenkins JVM configuration?**
- [ ] Configuration of Java Virtual Machine memory and options for Jenkins
- [ ] Configuration of Jenkins plugins
- [ ] Configuration of build agents
- [ ] Configuration of Jenkins UI

<details>
<summary>Answer</summary>
**Configuration of Java Virtual Machine memory and options for Jenkins**

JVM configuration involves setting Java heap size (`-Xms`, `-Xmx`), GC options, and other JVM parameters to optimize Jenkins performance. These are typically set in the Jenkins service configuration or startup scripts.
</details>

---

**25. What is the recommended JVM heap size for a medium Jenkins instance?**
- [ ] 256 MB
- [ ] 512 MB - 1 GB
- [ ] 2 GB - 8 GB
- [ ] 32 GB

<details>
<summary>Answer</summary>
**2 GB - 8 GB**

For a medium Jenkins instance (50-200 jobs, several agents), 2-8 GB of heap is typical. The exact size depends on the number of jobs, plugins, concurrent builds, and build history. Starting at 4 GB and monitoring is recommended.
</details>

---

**26. What is the purpose of the `-Dhudson.model.ParametersAction.keepUndefinedParameters=true` JVM option?**
- [ ] To keep undefined parameters in parameterized builds
- [ ] To allow passing undefined parameters to pipelines
- [ ] To suppress parameter validation warnings
- [ ] All of the above

<details>
<summary>Answer</summary>
**To allow passing undefined parameters to pipelines**

This JVM option controls whether Jenkins allows passing parameters that are not defined in the job configuration. Setting it to `true` allows passing extra parameters, useful when integrating with external systems.
</details>

---

**27. What is the purpose of the Jenkins `/administrativeMonitor/` URL?**
- [ ] To view and manage Jenkins administrative monitors
- [ ] To monitor administrative users
- [ ] To manage administrative plugins
- [ ] To view agent status

<details>
<summary>Answer</summary>
**To view and manage Jenkins administrative monitors**

The `/administrativeMonitor/` URL shows Jenkins' built-in health checks and warnings: outdated plugins, disk space low, security issues, upgrade recommendations, and other administrative concerns.
</details>

---

**28. What is the Jenkins log level configuration?**
- [ ] A setting to control the verbosity of Jenkins logging
- [ ] A setting to disable logging
- [ ] A setting to encrypt logs
- [ ] A setting to archive logs

<details>
<summary>Answer</summary>
**A setting to control the verbosity of Jenkins logging**

Jenkins log levels (FINE, FINER, FINEST for DEBUG, WARNING, SEVERE for ERROR) can be configured per logger (package or class) through the UI at `/log` or `/manage/log`. This helps debug specific issues without overwhelming logs.
</details>

---

**29. What is the purpose of the Jenkins `/script` URL?**
- [ ] To run Groovy scripts on the Jenkins controller
- [ ] To manage Pipeline scripts
- [ ] To install scripts
- [ ] To view build scripts

<details>
<summary>Answer</summary>
**To run Groovy scripts on the Jenkins controller**

The `/script` endpoint (Script Console) allows administrators to run arbitrary Groovy scripts on the Jenkins controller. This is a powerful debugging and administration tool but should be restricted to administrators due to security implications.
</details>

---

**30. What is the Jenkins Groovy Script Console used for?**
- [ ] Running ad-hoc administration commands
- [ ] Debugging Jenkins issues
- [ ] Automating bulk operations
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

The Script Console is used for: running ad-hoc administration (creating users, modifying jobs), debugging (inspecting variables, testing configurations), and automating bulk operations (mass job updates, credential management).
</details>

---

**31. What is the purpose of the `queue` command in the Jenkins Script Console?**
- [ ] To inspect the build queue
- [ ] To cancel queued builds
- [ ] To reorder the build queue
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

The `Jenkins.instance.queue` object in Script Console provides methods to inspect, cancel, and reorder items in the build queue. This is useful for managing long queues and debugging queue-related issues.
</details>

---

**32. What is the Jenkins Update Center?**
- [ ] A repository of Jenkins plugins and their metadata
- [ ] A system update tool
- [ ] A release management portal
- [ ] A plugin development platform

<details>
<summary>Answer</summary>
**A repository of Jenkins plugins and their metadata**

The Update Center is the central repository that provides plugin installation and update information to Jenkins instances. It contains metadata about available plugins, versions, dependencies, and compatibility information.
</details>

---

**33. Why should you keep Jenkins plugins updated?**
- [ ] Security fixes for known vulnerabilities
- [ ] Bug fixes and performance improvements
- [ ] New features and compatibility
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Regular plugin updates are critical for: security (many vulnerabilities are reported and fixed in plugins), stability (bug fixes), performance improvements, and compatibility with newer Jenkins core versions.
</details>

---

**34. What is the Plugin Manager's "Advanced" tab used for?**
- [ ] To manually upload plugin files (.hpi or .jpi)
- [ ] To configure update center URLs
- [ ] To view plugin release history
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

The Advanced tab in Plugin Manager provides: manual plugin upload (for offline installations), update center URL configuration (for proxies or mirrors), and plugin release history and dependency information.
</details>

---

**35. What is the purpose of the Jenkins CLI over SSH?**
- [ ] To manage Jenkins via SSH commands
- [ ] To connect agents via SSH
- [ ] To configure SSH credentials
- [ ] To encrypt Jenkins traffic

<details>
<summary>Answer</summary>
**To manage Jenkins via SSH commands**

The Jenkins CLI over SSH allows running Jenkins CLI commands through an SSH connection. This provides authenticated, encrypted access for administrative operations without exposing the HTTP CLI port.
</details>

---

**36. How do you configure Jenkins to use an external database?**
- [ ] By installing the Database plugin and configuring the JDBC connection
- [ ] By editing the Jenkins configuration file
- [ ] By setting environment variables
- [ ] External databases are not supported

<details>
<summary>Answer</summary>
**By installing the Database plugin and configuring the JDBC connection**

Jenkins can use external databases for specific purposes (like storing build data) through the Database plugin. The plugin is configured with JDBC connection details. General Jenkins configuration is not database-backed.
</details>

---

**37. What is the purpose of the `git-forensics` plugin?**
- [ ] To analyze Git history for code quality and change trends
- [ ] To track Git commits in builds
- [ ] To monitor Git server health
- [ ] To optimize Git operations

<details>
<summary>Answer</summary>
**To analyze Git history for code quality and change trends**

The Git Forensics plugin analyzes Git repository history to provide insights on code quality trends, file change frequencies, author statistics, and code churn metrics. This helps teams understand their development patterns.
</details>

---

**38. What is Jenkins performance signature?**
- [ ] A signature analysis of pipeline performance
- [ ] A benchmarking tool for Jenkins
- [ ] A plugin that tests Jenkins performance
- [ ] A method for optimizing build performance

<details>
<summary>Answer</summary>
**A signature analysis of pipeline performance**

Performance signatures track build duration trends over time, enabling detection of performance regressions. If a build takes significantly longer than its historical signature, Jenkins can flag it for investigation.
</details>

---

**39. What is the purpose of an incident response plan for Jenkins?**
- [ ] To provide procedures for responding to Jenkins failures
- [ ] To document recovery steps
- [ ] To outline communication protocols
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

An incident response plan for Jenkins should document: procedures for responding to controller failures, agent connectivity issues, build failures, security incidents, performance degradation, and the communication and escalation process.
</details>

---

**40. What is the SCM retry count in Jenkins?**
- [ ] The number of times Jenkins retries SCM operations on failure
- [ ] The number of SCM polling attempts
- [ ] The number of Git clone retries
- [ ] The number of plugin download retries

<details>
<summary>Answer</summary>
**The number of times Jenkins retries SCM operations on failure**

The SCM retry count (in job configuration) specifies how many times Jenkins retries SCM checkout operations if they fail due to network issues or transient errors. This improves build reliability.
</details>

---

**41. What is the purpose of the `quiet period` in Jenkins?**
- [ ] A delay before starting a build to allow more commits to accumulate
- [ ] A period when builds are paused
- [ ] A maintenance window
- [ ] A backup window

<details>
<summary>Answer</summary>
**A delay before starting a build to allow more commits to accumulate**

The quiet period is a configurable delay between when a build is triggered and when it actually starts. This allows developers to push multiple commits that are grouped into a single build, reducing build volume and improving efficiency.
</details>

---

**42. What is the Jenkins block build when upstream/downstream is building?**
- [ ] An option to prevent concurrent builds when dependencies are building
- [ ] An option to block all building
- [ ] An option to pause job scheduling
- [ ] An option to delay builds

<details>
<summary>Answer</summary>
**An option to prevent concurrent builds when dependencies are building**

This option prevents a job from starting a new build if an upstream (triggering) or downstream (triggered) job is currently building. This helps maintain build ordering and prevent conflicts.
</details>

---

**43. What is the best practice for Jenkins security patches?**
- [ ] Apply patches immediately after testing
- [ ] Wait for major releases
- [ ] Only patch when vulnerabilities are announced
- [ ] Never patch running Jenkins instances

<details>
<summary>Answer</summary>
**Apply patches immediately after testing**

Security patches should be applied as soon as possible after testing them in a non-production environment. Jenkins and its plugins have frequent security advisories, and timely patching is critical for security.
</details>

---

**44. What is the purpose of the Jenkins security advisories page?**
- [ ] To announce Jenkins security vulnerabilities and fixes
- [ ] To provide security best practices
- [ ] To list secure plugin versions
- [ ] To report security incidents

<details>
<summary>Answer</summary>
**To announce Jenkins security vulnerabilities and fixes**

The Jenkins project maintains a security advisory page that announces security vulnerabilities, affected versions, and fix versions. Administrators should monitor this page to stay informed about security patches.
</details>

---

**45. What is the purpose of the `configuration-as-code-groovy` plugin?**
- [ ] To allow using Groovy scripts in JCasC configurations
- [ ] To replace JCasC with Groovy
- [ ] To convert YAML to Groovy
- [ ] To validate JCasC configurations

<details>
<summary>Answer</summary>
**To allow using Groovy scripts in JCasC configurations**

The Configuration as Code Groovy plugin allows embedding Groovy scripts within JCasC YAML configurations. This enables configuration steps that cannot be expressed purely in YAML, while maintaining version control and reproducibility.
</details>

---

**46. What is the `support-core` plugin?**
- [ ] A plugin for generating Jenkins support bundles
- [ ] A plugin for managing support requests
- [ ] A plugin for troubleshooting Jenkins
- [ ] A plugin for user support

<details>
<summary>Answer</summary>
**A plugin for generating Jenkins support bundles**

The Support Core plugin generates comprehensive support bundles containing Jenkins configuration, logs, system information, and plugin details. These bundles are useful when seeking help from the Jenkins community or support team.
</details>

---

**47. What is the purpose of the `instance-identity` plugin?**
- [ ] To generate a unique identifier for the Jenkins instance
- [ ] To identify Jenkins agents
- [ ] To manage Jenkins users
- [ ] To authenticate Jenkins instances

<details>
<summary>Answer</summary>
**To generate a unique identifier for the Jenkins instance**

The Instance Identity plugin generates a cryptographic key pair and certificate that uniquely identifies the Jenkins instance. This is used for agent connections, CLI authentication, and securing inter-Jenkins communications.
</details>

---

**48. What is the recommended monitoring threshold for Jenkins disk space?**
- [ ] Alert when disk space falls below 20%
- [ ] Alert when disk space falls below 1 GB
- [ ] Alert when disk space falls below 5 GB
- [ ] Both A and C depending on instance size

<details>
<summary>Answer</summary>
**Both A and C depending on instance size**

Monitoring thresholds should be set based on instance size: alert when disk space is below 20% or when it falls below a specific threshold (e.g., 5 GB for smaller instances). Jenkins itself warns when disk space is low via administrative monitors.
</details>

---

**49. What is a Jenkins regression build?**
- [ ] A build that runs all previous tests to detect regressions
- [ ] A build that reverts changes
- [ ] A build that runs on older code
- [ ] A build that tests performance

<details>
<summary>Answer</summary>
**A build that runs all previous tests to detect regressions**

A regression build runs a comprehensive set of tests to check whether recent changes have broken previously working functionality. Jenkins can compare test results across builds using the JUnit plugin's trend reporting.
</details>

---

**50. What is the purpose of the `testng` plugin in Jenkins?**
- [ ] To publish TestNG test results and reports
- [ ] To run TestNG tests within Jenkins
- [ ] To configure TestNG settings
- [ ] To generate TestNG test data

<details>
<summary>Answer</summary>
**To publish TestNG test results and reports**

The TestNG plugin publishes TestNG test reports in Jenkins, providing detailed test results, failure analysis, and historical trends for TestNG-based test frameworks.
</details>

---

**51. What is the purpose of the `warnings-ng` plugin?**
- [ ] To collect and display compiler warnings and static analysis results
- [ ] To generate warning notifications
- [ ] To configure warning thresholds
- [ ] To monitor warning trends

<details>
<summary>Answer</summary>
**To collect and display compiler warnings and static analysis results**

The Warnings Next Generation plugin collects warnings from compilers, linters, and static analysis tools. It provides trend graphs, quality gates, and detailed warning information.
</details>

---

**52. What is the purpose of the `performance` plugin in Jenkins?**
- [ ] To collect and display performance test results (JMeter, Gatling, etc.)
- [ ] To monitor Jenkins performance
- [ ] To optimize build performance
- [ ] To benchmark Jenkins

<details>
<summary>Answer</summary>
**To collect and display performance test results (JMeter, Gatling, etc.)**

The Performance Plugin collects and displays performance test reports from tools like JMeter, Gatling, and Taurus. It provides trend graphs, performance thresholds, and failure analysis.
</details>

---

**53. What is the best practice for Jenkins access logs?**
- [ ] Enable Jenkins access logging and rotate logs
- [ ] Disable access logging for performance
- [ ] Store access logs in memory
- [ ] Access logs are not needed

<details>
<summary>Answer</summary>
**Enable Jenkins access logging and rotate logs**

Access logging should be enabled to track user activity, API usage, and security events. Logs should be rotated to manage disk space and archived for audit purposes. The audit trail plugin can provide detailed access logs.
</details>

---

**54. What is the purpose of the `audit-trail` plugin?**
- [ ] To track user actions and configuration changes in Jenkins
- [ ] To audit build logs
- [ ] To monitor system performance
- [ ] To track plugin usage

<details>
<summary>Answer</summary>
**To track user actions and configuration changes in Jenkins**

The Audit Trail plugin logs user actions such as job creation/deletion, configuration changes, plugin operations, and system administration activities. This provides accountability and helps with compliance requirements.
</details>

---

**55. What is the purpose of the `PrioritySorter` plugin?**
- [ ] To prioritize builds in the build queue
- [ ] To sort build artifacts
- [ ] To organize jobs by priority
- [ ] To prioritize agent selection

<details>
<summary>Answer</summary>
**To prioritize builds in the build queue**

The Priority Sorter plugin allows assigning priority values to jobs, influencing the order in which queued builds are assigned to executors. Higher priority jobs are dispatched before lower priority ones.
</details>

---

**56. What is the ESAPI (Enterprise Security API) in Jenkins?**
- [ ] A security framework used by Jenkins for input validation
- [ ] A plugin for security scanning
- [ ] A cryptography library
- [ ] An authentication provider

<details>
<summary>Answer</summary>
**A security framework used by Jenkins for input validation**

Jenkins uses the OWASP ESAPI library for input validation, output encoding, and other security functions. It helps prevent cross-site scripting (XSS) and other injection attacks in the Jenkins UI.
</details>

---

**57. What is the recommended approach for Jenkins capacity planning?**
- [ ] Monitor current usage and plan for growth
- [ ] Estimate based on number of users and jobs
- [ ] Plan for peak-load scenarios
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Capacity planning for Jenkins involves: monitoring current resource usage (CPU, memory, disk, agents), estimating growth in users, jobs, and build frequency, planning for peak loads (release days, sprint end), and proactively scaling resources.
</details>

---

> **Pro tip:** After completing this test, check your score per topic. If you scored below 70% in any area, review the corresponding chapters in this guide before attempting the next practice test.
