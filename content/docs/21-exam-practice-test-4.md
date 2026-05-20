---
title: "📝 Practice Test 4: Agents, Nodes & Distributed Builds"
weight: 21
bookToc: false
---

# Practice Test 4: Agents, Nodes & Distributed Builds

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
  <div class="exam-timer" data-minutes="60" data-test-id="exam-test-4">
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
    <input type="checkbox" class="test-complete-check" id="test-complete-4" data-test-id="exam-test-4">
    <label for="test-complete-4">✓ Mark test as completed</label>
  </div>
</div>

**57 Questions | Duration: 60 minutes | Focus: Agents, nodes, distributed builds, Docker & Kubernetes integration**

---

## Questions

**1. What is a Jenkins agent?**
- [ ] A plugin that manages builds
- [ ] A worker machine that executes build jobs assigned by the controller
- [ ] A Jenkins job type
- [ ] A build artifact repository

<details>
<summary>Answer</summary>
**A worker machine that executes build jobs assigned by the controller**

A Jenkins agent (also called a node or slave) is a machine that receives work from the Jenkins controller. The controller distributes build jobs to agents, which execute the actual build steps and report results back.
</details>

---

**2. How does a Jenkins agent connect to the controller?**
- [ ] Via HTTP/HTTPS
- [ ] Via SSH
- [ ] Via JNLP (Java Web Start)
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Jenkins agents can connect to the controller using various protocols: SSH (most common for Linux agents), JNLP (Java Web Start, TCP-based), or HTTP/HTTPS. The choice depends on network topology and security requirements.
</details>

---

**3. What is the difference between a permanent agent and a cloud agent?**
- [ ] Permanent agents are always running; cloud agents are provisioned on-demand
- [ ] Permanent agents are faster
- [ ] Cloud agents are more secure
- [ ] There is no difference

<details>
<summary>Answer</summary>
**Permanent agents are always running; cloud agents are provisioned on-demand**

Permanent agents are machines that are always available and manually configured. Cloud agents are dynamically provisioned from cloud providers (AWS, Azure, GCP) or container platforms (Docker, Kubernetes) when needed and terminated when idle.
</details>

---

**4. What is the Jenkins agent.jar file used for?**
- [ ] To install Jenkins on the controller
- [ ] To connect an agent to the Jenkins controller
- [ ] To manage Jenkins plugins
- [ ] To run Jenkins builds locally

<details>
<summary>Answer</summary>
**To connect an agent to the Jenkins controller**

The `agent.jar` file (now `remoting.jar`) is the agent software that runs on worker machines to establish and maintain the connection with the Jenkins controller. It receives build commands and reports results.
</details>

---

**5. What is the default agent communication port for JNLP?**
- [ ] 8080
- [ ] 50000
- [ ] 443
- [ ] 22

<details>
<summary>Answer</summary>
**50000**

Jenkins uses TCP port 50000 by default for JNLP-based agent connections. This port must be accessible from agent machines to establish the connection. The port can be changed in Jenkins global security settings.
</details>

---

**6. What is a cloud agent in Jenkins?**
- [ ] An agent that runs on the Jenkins controller
- [ ] A dynamically provisioned agent from cloud infrastructure
- [ ] A web-based agent management tool
- [ ] A monitoring service

<details>
<summary>Answer</summary>
**A dynamically provisioned agent from cloud infrastructure**

Cloud agents are provisioned on-demand from cloud providers (AWS EC2, Azure VMs, Google Cloud), container platforms (Docker, Kubernetes), or other dynamic environments. They are created when builds are queued and terminated after completion.
</details>

---

**7. What is the purpose of agent labels?**
- [ ] To identify agent types and capabilities for job matching
- [ ] To display agent status
- [ ] To organize agents in the UI
- [ ] To encrypt agent communication

<details>
<summary>Answer</summary>
**To identify agent types and capabilities for job matching**

Agent labels are user-defined tags assigned to agents. Jobs can specify which labels they require (e.g., `linux`, `docker`, `gpu`), and Jenkins only runs those jobs on agents with matching labels.
</details>

---

**8. How do you restrict a pipeline to run on a specific agent?**
- [ ] `agent { label 'linux' }`
- [ ] `agent { any }`
- [ ] `agent { run 'linux' }`
- [ ] `restrict('linux')`

<details>
<summary>Answer</summary>
**`agent { label 'linux' }`**

In Declarative Pipeline, use `agent { label 'linux' }` to restrict the pipeline or stage to agents with the matching label. Labels are case-sensitive.
</details>

---

**9. What is an executor in Jenkins?**
- [ ] A tool that executes commands on the controller
- [ ] A slot on a node that can run one build at a time
- [ ] A Jenkins plugin for task execution
- [ ] A build scheduling service

<details>
<summary>Answer</summary>
**A slot on a node that can run one build at a time**

An executor represents a unit of concurrency on a Jenkins node. Each executor can run one build at a time. The number of executors on a node determines how many concurrent builds that node can handle.
</details>

---

**10. How many executors should you configure on the Jenkins controller?**
- [ ] As many as possible
- [ ] 0 (zero)
- [ ] 2-4
- [ ] 1

<details>
<summary>Answer</summary>
**0 (zero)**

Best practice is to set the controller's executor count to 0. This prevents build jobs from running on the controller, reserving its CPU and memory for system operations (UI, job scheduling, plugin management) and improving security.
</details>

---

**11. What does the `agent { docker { image 'maven:3.8' } }` directive do?**
- [ ] Installs Maven on the agent
- [ ] Runs the pipeline stage inside a Docker container with the Maven image
- [ ] Downloads the Maven image to the agent
- [ ] Configures the agent to use Maven

<details>
<summary>Answer</summary>
**Runs the pipeline stage inside a Docker container with the Maven image**

This directive instructs Jenkins to run the pipeline or stage inside a Docker container using the specified image. Jenkins pulls the image, starts the container, mounts the workspace, and executes pipeline steps inside it.
</details>

---

**12. What is a Jenkins agent SSH private key used for?**
- [ ] To encrypt build artifacts
- [ ] To authenticate the controller with agents over SSH
- [ ] To sign Jenkins plugins
- [ ] To encrypt user passwords

<details>
<summary>Answer</summary>
**To authenticate the controller with agents over SSH**

When using the SSH method to connect agents, the controller authenticates to the agent using an SSH key pair. The controller holds the private key, while the public key is installed on the agent machine.
</details>

---

**13. What is the difference between an agent and a node in Jenkins?**
- [ ] They are the same concept
- [ ] An agent is a type of node
- [ ] A node includes the controller; an agent is a worker only
- [ ] A node is virtual; an agent is physical

<details>
<summary>Answer</summary>
**A node includes the controller; an agent is a worker only**

In Jenkins terminology, "node" refers to any machine in the Jenkins environment (including the controller), while "agent" refers specifically to worker machines that execute builds. The controller is a node but not an agent.
</details>

---

**14. What is the purpose of the built-in node in Jenkins?**
- [ ] To run builds by default
- [ ] To represent the Jenkins controller as a node
- [ ] To manage agent connections
- [ ] To store build artifacts

<details>
<summary>Answer</summary>
**To represent the Jenkins controller as a node**

The built-in node represents the Jenkins controller itself in the node list. By default, it has 2 executors. Best practice sets its executors to 0 to prevent builds from running on the controller.
</details>

---

**15. How do you add a new agent in Jenkins?**
- [ ] Jenkins > Manage Jenkins > Manage Nodes and Clouds > New Node
- [ ] Jenkins > New Agent
- [ ] Edit configuration files
- [ ] Install an agent plugin

<details>
<summary>Answer</summary>
**Jenkins > Manage Jenkins > Manage Nodes and Clouds > New Node**

New agents are added through the Jenkins UI under Manage Jenkins > Manage Nodes and Clouds, then clicking "New Node". You configure the agent name, remote root directory, labels, launch method, and number of executors.
</details>

---

**16. What is the `remote root directory` in agent configuration?**
- [ ] The location of Jenkins system files on the agent
- [ ] The workspace directory on the agent
- [ ] The agent installation directory
- [ ] The Jenkins home directory on the controller

<details>
<summary>Answer</summary>
**The workspace directory on the agent**

The remote root directory is the base directory on the agent where Jenkins workspaces and build artifacts are stored. Each job gets a subdirectory within this path for its workspace.
</details>

---

**17. What is the availability setting for an agent?**
- [ ] How often the agent checks for updates
- [ ] When the agent should be available for builds
- [ ] The agent's network availability
- [ ] The agent's storage availability

<details>
<summary>Answer</summary>
**When the agent should be available for builds**

The availability setting determines when an agent is online and available to execute builds. Options include "Keep this agent online as much as possible", "Take this agent online and offline at specific times", or "Take this agent online when in demand, and offline when idle".
</details>

---

**18. What is a Docker agent in Jenkins?**
- [ ] An agent that runs Docker commands
- [ ] A dynamically provisioned agent running as a Docker container
- [ ] A Docker plugin
- [ ] A Docker registry

<details>
<summary>Answer</summary>
**A dynamically provisioned agent running as a Docker container**

A Docker agent is a Jenkins agent that runs as a Docker container. The Docker Plugin dynamically creates containers as agents when builds are queued and terminates them when idle. Each container provides a clean, isolated environment.
</details>

---

**19. How does Jenkins integrate with Kubernetes for agent provisioning?**
- [ ] By using the Kubernetes plugin to dynamically create pods as agents
- [ ] By running Jenkins inside Kubernetes
- [ ] By deploying builds to Kubernetes
- [ ] By monitoring Kubernetes clusters

<details>
<summary>Answer</summary>
**By using the Kubernetes plugin to dynamically create pods as agents**

The Kubernetes Plugin allows Jenkins controllers running outside or inside Kubernetes to dynamically create agent pods in the Kubernetes cluster. Each pod serves as an agent, providing ephemeral build environments with full isolation.
</details>

---

**20. What is a Kubernetes pod template in Jenkins?**
- [ ] A template for generating Kubernetes configurations
- [ ] A definition of the agent pod's containers, volumes, and resources
- [ ] A Jenkins job template
- [ ] A build pipeline template

<details>
<summary>Answer</summary>
**A definition of the agent pod's containers, volumes, and resources**

A Kubernetes pod template defines how Jenkins should create agent pods — including the containers to run, their images, resource limits, environment variables, volumes, and other Kubernetes pod specifications.
</details>

---

**21. What is the purpose of the `jnlp` container in a Kubernetes pod template?**
- [ ] To execute build steps
- [ ] To establish the JNLP connection between the agent and controller
- [ ] To store build artifacts
- [ ] To manage pod networking

<details>
<summary>Answer</summary>
**To establish the JNLP connection between the agent and controller**

The `jnlp` container in a Kubernetes pod template runs the Jenkins remoting agent that connects to the controller. Additional containers can run alongside it for build tools (Maven, Node, Docker), and pipeline steps can run in specific containers.
</details>

---

**22. How do you specify a container in a Kubernetes pod within a Pipeline?**
- [ ] `container('maven') { ... }`
- [ ] `pod('maven') { ... }`
- [ ] `docker('maven') { ... }`
- [ ] `inside('maven') { ... }`

<details>
<summary>Answer</summary>
**`container('maven') { ... }`**

In a Kubernetes pod template, the `container('name')` step allows running specific pipeline steps inside a particular container of the pod. This enables using different tools from different containers within the same pipeline stage.
</details>

---

**23. What is the advantage of using ephemeral agents?**
- [ ] They are faster
- [ ] They provide clean, isolated environments for each build
- [ ] They require less configuration
- [ ] They use less disk space

<details>
<summary>Answer</summary>
**They provide clean, isolated environments for each build**

Ephemeral agents (like Docker containers or Kubernetes pods) are created fresh for each build and terminated afterward. This ensures every build starts with a clean environment, eliminating issues from leftover files, cached state, or configuration drift.
</details>

---

**24. What is the purpose of the `idleMinutes` setting in cloud agent configuration?**
- [ ] The number of minutes before an idle agent is terminated
- [ ] The number of minutes an agent takes to start
- [ ] The maximum build duration
- [ ] The agent's uptime requirement

<details>
<summary>Answer</summary>
**The number of minutes before an idle agent is terminated**

The `idleMinutes` setting specifies how long a cloud-provisioned agent should remain online after completing its build before being terminated. This prevents premature termination if another build might use the same agent.
</details>

---

**25. How does Jenkins manage agent security?**
- [ ] Through agent-to-controller access control
- [ ] Through encrypted communication channels
- [ ] Through credential isolation
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Jenkins provides multiple security layers for agents: agent-to-controller access control prevents agents from accessing sensitive controller files, communication channels can be encrypted (HTTPS/SSH), and credentials are isolated from agent processes.
</details>

---

**26. What happens when an agent goes offline unexpectedly?**
- [ ] The controller shuts down
- [ ] Running builds on that agent are interrupted and marked as failed
- [ ] The agent automatically restarts
- [ ] Builds are paused until the agent reconnects

<details>
<summary>Answer</summary>
**Running builds on that agent are interrupted and marked as failed**

When an agent goes offline unexpectedly (network failure, crash, shutdown), any builds running on that agent are forcibly terminated and marked as failed. Queued builds remain in the queue for other agents.
</details>

---

**27. What is the function of the `agent tunnel` in Jenkins?**
- [ ] A VPN connection for agents
- [ ] A tunnel through a proxy for agent communication
- [ ] A secure communication channel
- [ ] A monitoring tunnel

<details>
<summary>Answer</summary>
**A tunnel through a proxy for agent communication**

The agent tunnel configuration allows agents to connect to the Jenkins controller through a proxy or firewall. The tunnel specifies an alternative host and port for agents to use when direct connection to the controller is not possible.
</details>

---

**28. How do you enable agent-to-controller security?**
- [ ] Under Jenkins > Configure Global Security
- [ ] By installing a security plugin
- [ ] By configuring agent launch methods
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Agent-to-controller security is configured through multiple mechanisms: Global Security settings control the access control whitelist, the Security plugin provides advanced options, and agent launch methods determine communication channels.
</details>

---

**29. What is the purpose of the `java -jar agent.jar -jnlpUrl` command?**
- [ ] To start the Jenkins controller
- [ ] To connect an agent to the controller via JNLP
- [ ] To run a Jenkins build
- [ ] To install Jenkins plugins

<details>
<summary>Answer</summary>
**To connect an agent to the controller via JNLP**

The command `java -jar agent.jar -jnlpUrl <controller-url>/computer/<agent-name>/agent.jnlp` launches the Jenkins agent process and connects it to the controller using the JNLP protocol. This is common for Windows or manually managed agents.
</details>

---

**30. What is the function of the `webSocket` agent connection method?**
- [ ] An alternative way for agents to connect using WebSocket protocol
- [ ] A browser-based agent connection
- [ ] A monitoring tool
- [ ] A plugin management tool

<details>
<summary>Answer</summary>
**An alternative way for agents to connect using WebSocket protocol**

The WebSocket agent connection method allows agents to connect to the controller using the WebSocket protocol. This is useful when agents are behind firewalls or proxies that don't allow the standard TCP/JNLP connections.
</details>

---

**31. What is an agent's workspace?**
- [ ] The Jenkins home directory
- [ ] A directory on the agent where build source code and artifacts reside
- [ ] The controller's build directory
- [ ] A shared network drive

<details>
<summary>Answer</summary>
**A directory on the agent where build source code and artifacts reside**

An agent's workspace is a directory on the agent's filesystem where the source code is checked out and build operations are performed. Each job typically gets a subdirectory named after the job within the workspace root.
</details>

---

**32. What is the best practice for allocating executors per agent?**
- [ ] One executor per CPU core
- [ ] Two executors per agent
- [ ] As many as the agent's resources allow
- [ ] Match the number of executors to the number of concurrent builds desired

<details>
<summary>Answer</summary>
**Match the number of executors to the number of concurrent builds desired**

The number of executors should be set based on the agent's CPU cores, memory, and I/O capacity. A common starting point is the number of CPU cores, adjusted based on how resource-intensive builds are.
</details>

---

**33. What is the purpose of the "Use WebSocket" option in agent configuration?**
- [ ] To enable encrypted communication
- [ ] To use WebSocket protocol for agent-controller connection
- [ ] To enable real-time monitoring
- [ ] To support browser-based agents

<details>
<summary>Answer</summary>
**To use WebSocket protocol for agent-controller connection**

The "Use WebSocket" option switches agent communication from the default TCP/JNLP protocol to WebSocket. This is beneficial when agents must connect through web proxies or reverse proxies that only support HTTP/HTTPS traffic.
</details>

---

**34. How do you configure Jenkins to run Docker commands on an agent?**
- [ ] By installing Docker on the agent and adding the `docker` label
- [ ] By using the Docker plugin
- [ ] By configuring a Docker cloud
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Docker support requires Docker installed on the agent, appropriate labels for job matching, the Docker plugin for advanced features, and optionally a Docker cloud configuration for provisioning Docker-based agents.
</details>

---

**35. What is the Docker Pipeline plugin?**
- [ ] A plugin that builds Docker images
- [ ] A plugin that integrates Docker operations into Jenkins Pipelines
- [ ] A plugin that manages Docker registries
- [ ] A plugin that monitors Docker containers

<details>
<summary>Answer</summary>
**A plugin that integrates Docker operations into Jenkins Pipelines**

The Docker Pipeline plugin provides pipeline steps for building Docker images, pushing them to registries, running containers, and using Docker containers as build environments directly within Jenkins Pipelines.
</details>

---

**36. What does the `docker.build()` step do in a Pipeline?**
- [ ] Installs Docker on the agent
- [ ] Builds a Docker image from a Dockerfile
- [ ] Starts a Docker container
- [ ] Downloads a Docker image

<details>
<summary>Answer</summary>
**Builds a Docker image from a Dockerfile**

The `docker.build()` step builds a Docker image using a Dockerfile in the workspace. It accepts the image name and optionally the Dockerfile path. The built image is available locally on the agent.
</details>

---

**37. What does the `docker.withRegistry()` step do?**
- [ ] Configures a Docker registry for authentication and push/pull operations
- [ ] Starts a local Docker registry
- [ ] Syncs Docker images with a registry
- [ ] Monitors Docker registry activity

<details>
<summary>Answer</summary>
**Configures a Docker registry for authentication and push/pull operations**

`docker.withRegistry()` sets up a Docker registry URL and credentials for the enclosed block. It enables authenticated access for pushing built images and pulling private images from the registry.
</details>

---

**38. How do you push a Docker image in a Pipeline?**
- [ ] `docker.push('myimage:latest')`
- [ ] `dockerImage.push()`
- [ ] Both A and B depending on the use of `docker.build()`
- [ ] `pushDocker 'myimage:latest'`

<details>
<summary>Answer</summary>
**Both A and B depending on the use of `docker.build()`**

When using `docker.build()`, the return value is a Docker image object with a `.push()` method. Alternatively, `docker.push()` can push a named image directly. Both push the image to a configured registry.
</details>

---

**39. What is the purpose of the `inside` step with Docker Pipeline?**
- [ ] To run pipeline steps inside a Docker container
- [ ] To inspect a Docker container
- [ ] To SSH into a container
- [ ] To execute commands on the Docker host

<details>
<summary>Answer</summary>
**To run pipeline steps inside a Docker container**

The `docker.image('...').inside()` step runs the enclosed pipeline steps inside a Docker container. Jenkins mounts the workspace, sets up the environment, and executes steps within the container, providing full isolation.
</details>

---

**40. What is the purpose of the `containerLog` step in Kubernetes plugin?**
- [ ] To write logs to a container
- [ ] To retrieve logs from a specific container in a pod
- [ ] To configure container logging
- [ ] To send logs to stdout

<details>
<summary>Answer</summary>
**To retrieve logs from a specific container in a pod**

The `containerLog` step (from the Kubernetes plugin) retrieves the console log of a specific container within a Kubernetes pod. This is useful for debugging container startup issues or checking sidecar container output.
</details>

---

**41. What is the difference between `docker.image('node').inside()` and `agent { docker { image 'node' } }`?**
- [ ] `inside()` runs within a stage; `agent docker` runs at the pipeline level
- [ ] They are equivalent
- [ ] `inside()` requires a Docker registry
- [ ] `agent docker` is for declarative; `inside()` is for scripted

<details>
<summary>Answer</summary>
**`inside()` runs within a stage; `agent docker` runs at the pipeline level**

`agent { docker { ... } }` is Declarative syntax that sets the Docker agent for a pipeline or stage. `docker.image().inside()` is a pipeline step (usable in both Declarative and Scripted) that runs specific steps inside a container while the stage may have a different agent.
</details>

---

**42. How do you specify Docker arguments like environment variables?**
- [ ] `agent { docker { image 'node'; args '-e MY_VAR=value' } }`
- [ ] `docker.image('node').inside('-e MY_VAR=value')`
- [ ] Both A and B
- [ ] Docker arguments cannot be specified

<details>
<summary>Answer</summary>
**Both A and B**

Docker arguments can be specified in Declarative Pipeline with `args` in the `docker` agent section, or in Scripted Pipeline by passing arguments to the `.inside()` method. Arguments include `-e` for env vars, `-v` for volumes, and `--network` for network configuration.
</details>

---

**43. What is the purpose of the `agent { dockerfile true }` directive?**
- [ ] To build an image from a Dockerfile and use it as the agent
- [ ] To lint the Dockerfile
- [ ] To copy the Dockerfile to the workspace
- [ ] To push the Dockerfile to a registry

<details>
<summary>Answer</summary>
**To build an image from a Dockerfile and use it as the agent**

`agent { dockerfile true }` instructs Jenkins to build a Docker image from the Dockerfile in the workspace and then use that image as the build agent. This is useful when the build environment is defined entirely in a Dockerfile.
</details>

---

**44. What is the benefit of using Docker containers as build agents?**
- [ ] Consistent, reproducible build environments
- [ ] No need to maintain agent software
- [ ] Isolation from other builds
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Docker-based agents provide consistent build environments (eliminating "it works on my machine" issues), require no persistent agent management, and provide complete isolation between builds. Each build gets a fresh container.
</details>

---

**45. What is a Kubernetes pod in Jenkins context?**
- [ ] A Jenkins job
- [ ] A group of containers that form the build environment
- [ ] A Jenkins controller
- [ ] A repository of build artifacts

<details>
<summary>Answer</summary>
**A group of containers that form the build environment**

In the Kubernetes plugin, a pod is a group of one or more containers that collectively serve as a Jenkins agent. The pod includes a JNLP container for controller communication and one or more build containers with the required tools.
</details>

---

**46. What is the `yaml` option in Kubernetes pod templates?**
- [ ] A YAML configuration file for the pod template
- [ ] A simplified way to define the pod using raw Kubernetes YAML
- [ ] A YAML validator
- [ ] A YAML parser

<details>
<summary>Answer</summary>
**A simplified way to define the pod using raw Kubernetes YAML**

The `yaml` option in pod template configuration allows defining the entire pod specification as raw Kubernetes YAML. This provides maximum flexibility and supports any Kubernetes pod specification features.
</details>

---

**47. How do you define an environment variable in a Kubernetes pod template?**
- [ ] `envVars: [envVar(key: 'MY_VAR', value: 'myval')]`
- [ ] In the pod's YAML configuration
- [ ] Both A and B
- [ ] Environment variables cannot be defined in pod templates

<details>
<summary>Answer</summary>
**Both A and B**

Environment variables can be defined in Kubernetes pod templates using the `envVars` configuration option in the Jenkins UI or plugin configuration, or directly in the YAML specification for maximum flexibility.
</details>

---

**48. What is the purpose of the `podRetention` setting in the Kubernetes plugin?**
- [ ] To control how long a pod remains after a build completes
- [ ] To configure pod data retention
- [ ] To store pod logs
- [ ] To backup pod configurations

<details>
<summary>Answer</summary>
**To control how long a pod remains after a build completes**

`podRetention` controls the lifecycle of Kubernetes pods after builds complete. Options include `always` (pod stays for debugging), `never` (pod is deleted immediately), `onFailure` (pod stays if build fails), and `default` (based on idleMinutes).
</details>

---

**49. What is a Docker host validation strategy?**
- [ ] A strategy to validate Docker host connectivity
- [ ] A strategy to check Docker image integrity
- [ ] A strategy for verifying Docker daemon availability
- [ ] A security validation mechanism

<details>
<summary>Answer</summary>
**A strategy for verifying Docker daemon availability**

The Docker plugin's host validation strategy checks whether the Docker daemon on an agent is accessible and responding. If the validation fails, Jenkins can take configured actions like marking the agent offline.
</details>

---

**50. How do you use Docker Compose in a Jenkins Pipeline?**
- [ ] Using the `docker-compose` command via `sh` step
- [ ] Using the Docker Compose plugin
- [ ] Using a custom pipeline step
- [ ] Docker Compose is not supported

<details>
<summary>Answer</summary>
**Using the `docker-compose` command via `sh` step**

Docker Compose can be used in Jenkins Pipelines by executing `docker-compose` commands through the `sh` step. There are also community plugins that provide dedicated compose steps, but the `sh` approach is most common.
</details>

---

**51. What is the optimal way to cache Docker layers in Jenkins?**
- [ ] By using the `--cache-from` option in Docker builds
- [ ] By sharing the Docker cache volume across builds
- [ ] By using a Docker registry as a cache
- [ ] All of the above

<details>
<summary>Answer</summary>
**All of the above**

Docker layer caching can be optimized through `--cache-from` with a previously built image, mounting a shared Docker cache volume on persistent agents, or configuring a Docker registry pull-through cache.
</details>

---

**52. What is the Jenkins Kubernetes plugin's `slaveConnectTimeout` setting?**
- [ ] The timeout for agent connection
- [ ] The timeout for agent registration
- [ ] The maximum time to wait for a pod to connect as a Jenkins agent
- [ ] The timeout for agent disconnection

<details>
<summary>Answer</summary>
**The maximum time to wait for a pod to connect as a Jenkins agent**

`slaveConnectTimeout` (agent connection timeout) specifies how long Jenkins waits for a Kubernetes pod to establish the JNLP connection before considering the pod creation failed. Default is typically 100-600 seconds.
</details>

---

**53. What is the purpose of the `stages` in a DevOps pipeline?**
- [ ] To build code
- [ ] To represent logical phases of the CI/CD workflow
- [ ] To test the application
- [ ] To deploy to production

<details>
<summary>Answer</summary>
**To represent logical phases of the CI/CD workflow**

Stages in a DevOps pipeline represent the logical phases that code passes through: build, unit test, integration test, staging deployment, security scan, and production deployment. Each stage provides a clear checkpoint for validation.
</details>

---

**54. What is the purpose of the `dependsOn` feature in Kubernetes pod templates?**
- [ ] To specify pod dependencies
- [ ] To define container startup order within a pod
- [ ] To define job dependencies
- [ ] To manage plugin dependencies

<details>
<summary>Answer</summary>
**To define container startup order within a pod**

The `dependsOn` feature in Kubernetes pod templates allows specifying that one container should start only after another has started. This is useful when sidecar containers (like databases) must be available before build containers.
</details>

---

**55. What is the `keepLongRunningAgent` option in Kubernetes plugin?**
- [ ] Keeps the agent pod running for a specified duration
- [ ] Prevents agent pod termination for long-running tasks
- [ ] Keeps the agent pod after the build for debugging
- [ ] Extends the agent's idle timeout

<details>
<summary>Answer</summary>
**Keeps the agent pod after the build for debugging**

`keepLongRunningAgent` (or `alwaysConnect`) keeps the Kubernetes agent pod running even after the build completes, allowing developers to inspect the environment. This is useful for debugging but should be used judiciously.
</details>

---

**56. What is the Jenkinsfile's `pipeline` block for a Kubernetes-deployed application?**
- [ ] Build, test, Dockerize, deploy to Kubernetes
- [ ] Build only
- [ ] Deploy only
- [ ] Test only

<details>
<summary>Answer</summary>
**Build, test, Dockerize, deploy to Kubernetes**

A complete Jenkinsfile for a Kubernetes-deployed application typically includes stages for building the application, running tests, building a Docker image, pushing to a registry, and deploying the updated image to the Kubernetes cluster.
</details>

---

**57. What is the best practice for cleaning up resources in container-based Jenkins agents?**
- [ ] Use `post { always { cleanWs() } }` to clean workspace
- [ ] Use ephemeral agents that are deleted after each build
- [ ] Both A and B
- [ ] No cleanup is necessary

<details>
<summary>Answer</summary>
**Both A and B**

Best practice combines cleanup strategies: use `post { always { cleanWs() } }` in pipelines to clean workspace, and use ephemeral agents (Docker containers or Kubernetes pods) that are automatically destroyed after each build.
</details>

---

> **Pro tip:** After completing this test, check your score per topic. If you scored below 70% in any area, review the corresponding chapters in this guide before attempting the next practice test.
