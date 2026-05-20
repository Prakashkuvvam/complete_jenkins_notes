---
title: "Ch 11: Docker & Kubernetes Integration"
weight: 11
---

# Docker & Kubernetes Integration

## Learning Objectives

After reading this chapter, you will be able to:
- Build and push Docker images from Jenkins pipelines
- Use Docker containers as build agents
- Configure the Kubernetes plugin for dynamic agent provisioning
- Run pipelines inside Kubernetes pods
- Use container-based testing patterns
- Integrate with Helm for Kubernetes deployments

---

## 11.1 Docker in Jenkins Pipelines

### Docker Pipeline Plugin

The **Docker Pipeline** plugin provides first-class Docker support in Jenkinsfiles.

```groovy
pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build('myapp:latest')
                    
                    // Alternative: build with args
                    def image2 = docker.build(
                        'myapp:latest',
                        '--build-arg VERSION=1.0 -f docker/Dockerfile.prod .'
                    )
                }
            }
        }
    }
}
```

### Using Docker as a Build Agent

```groovy
pipeline {
    // Run pipeline inside a Docker container
    agent {
        docker { 
            image 'node:18-alpine'
            label 'docker'
            args '--network host'
            reuseNode true
        }
    }
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

### Publishing to Docker Registry

```groovy
pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        IMAGE_TAG = "${DOCKER_REGISTRY}/myapp:${BUILD_NUMBER}"
    }
    stages {
        stage('Build & Push') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'registry-creds') {
                        def img = docker.build(IMAGE_TAG)
                        img.push()
                        img.push('latest')
                    }
                }
            }
        }
    }
}
```

---

## 11.2 Docker Compose for Integration Tests

```groovy
pipeline {
    agent any
    stages {
        stage('Integration Test') {
            steps {
                script {
                    dockerCompose([
                        dockerComposeFile: 'docker-compose.test.yml',
                        projectName: "test-${BUILD_NUMBER}",
                        build: true,
                        upCommand: {
                            dockerCompose.up('-d')
                        },
                        downCommand: {
                            dockerCompose.down('-v')
                        }
                    ])
                }
                
                // Wait for services
                sh 'sleep 10'
                
                // Run tests against the composition
                sh 'curl -f http://localhost:8080/health'
            }
        }
    }
    
    post {
        always {
            // Clean up containers
            sh 'docker-compose -f docker-compose.test.yml -p test-* down -v'
        }
    }
}
```

### Docker Compose with Services

```yaml
# docker-compose.test.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
      - redis
    environment:
      - DB_URL=jdbc:postgresql://db:5432/testdb
      - REDIS_HOST=redis
      
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: testdb
      POSTGRES_PASSWORD: test
      
  redis:
    image: redis:7-alpine
```

---

## 11.3 Docker-in-Docker (DinD)

For building Docker images inside Docker containers:

```groovy
pipeline {
    agent {
        docker {
            image 'docker:20.10-dind'
            args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t myapp:latest .
                    docker save myapp:latest > myapp.tar
                '''
            }
        }
    }
}
```

---

## 11.4 Kubernetes Plugin

### Installing and Configuring

1. Install **Kubernetes** plugin
2. Dashboard → Manage Jenkins → Clouds → Add a new cloud → **Kubernetes**

```
Name: k8s-cloud
Kubernetes URL: https://kubernetes.default.svc
Kubernetes namespace: jenkins
Credentials: kubeconfig
Jenkins URL: http://jenkins.jenkins.svc.cluster.local:8080
Pod Retention: Never
☑ WebSocket for connection
```

### Pod Template Definitions

#### Simple Pod Template

```groovy
pipeline {
    agent {
        kubernetes {
            label 'build-pod'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command:
    - cat
    tty: true
  - name: docker
    image: docker:20.10-dind
    command:
    - cat
    tty: true
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
    securityContext:
      privileged: true
'''
            defaultContainer 'maven'
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Dockerize') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:latest .'
                }
            }
        }
    }
}
```

#### Advanced Pod with Sidecars

```groovy
pipeline {
    agent {
        kubernetes {
            label 'test-pod'
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-build
spec:
  serviceAccountName: jenkins
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
  - name: app
    image: node:18-alpine
    command:
    - cat
    tty: true
  - name: selenium
    image: selenium/standalone-chrome:latest
    env:
    - name: SE_VNC_PASSWORD
      value: "password"
  - name: db
    image: postgres:15-alpine
    env:
    - name: POSTGRES_DB
      value: test
    - name: POSTGRES_PASSWORD
      value: test
'''
            defaultContainer 'app'
        }
    }
    stages {
        stage('Test with Services') {
            steps {
                container('app') {
                    sh '''
                        npm ci
                        # DB and Selenium are available as localhost
                        DATABASE_URL=postgres://postgres:test@localhost:5432/test \\
                        npm test
                    '''
                }
            }
        }
    }
}
```

---

## 11.5 Container-Based Testing Patterns

### Multi-Container Integration Tests

```groovy
stage('Integration Tests') {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    image: myapp:${BUILD_NUMBER}
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 5
  - name: tests
    image: node:18-alpine
    command:
    - cat
    tty: true
  - name: db
    image: postgres:15-alpine
    env:
    - name: POSTGRES_DB
      value: appdb
    - name: POSTGRES_PASSWORD
      value: test
'''
            defaultContainer 'tests'
        }
    }
    stages {
        stage('Wait for App') {
            steps {
                // Wait for the app container to be ready
                sh '''
                    for i in $(seq 1 30); do
                        curl -f http://localhost:8080/health && break || sleep 2
                    done
                '''
            }
        }
        stage('Run API Tests') {
            steps {
                sh 'npm run test:api'
            }
        }
    }
}
```

---

## 11.6 Helm Integration

### Deploying with Helm

```groovy
pipeline {
    agent any
    environment {
        K8S_NAMESPACE = 'myapp'
        HELM_RELEASE = "myapp-${BUILD_NUMBER}"
    }
    stages {
        stage('Build & Push Image') {
            steps {
                script {
                    docker.withRegistry('', 'docker-creds') {
                        docker.build("myapp:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy with Helm') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh '''
                        helm upgrade --install ${HELM_RELEASE} ./chart \\
                            --namespace ${K8S_NAMESPACE} \\
                            --set image.tag=${BUILD_NUMBER} \\
                            --set replicaCount=3 \\
                            --wait \\
                            --timeout 5m
                    '''
                }
            }
        }
        
        stage('Smoke Test') {
            steps {
                sh '''
                    # Get the service URL
                    URL=$(kubectl get svc -n ${K8S_NAMESPACE} \\
                        -l app.kubernetes.io/instance=${HELM_RELEASE} \\
                        -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}')
                    
                    # Wait for the service to be ready
                    for i in $(seq 1 30); do
                        curl -f http://${URL}:8080/health && break || sleep 10
                    done
                    
                    echo "Smoke test passed!"
                '''
            }
        }
    }
    
    post {
        success {
            // Promote the release (remove old releases)
            sh """
                helm list -n ${K8S_NAMESPACE} --short | \\
                head -n -5 | \\
                xargs -r helm delete -n ${K8S_NAMESPACE}
            """
        }
        failure {
            // Rollback on failure
            sh "helm rollback ${HELM_RELEASE} -n ${K8S_NAMESPACE}"
        }
    }
}
```

---

## 11.7 Best Practices

### Docker Best Practices

| Practice | Benefit |
|----------|---------|
| Use specific image tags, not `latest` | Reproducible builds |
| Clean up unused images/containers | Save disk space |
| Use `.dockerignore` | Faster builds, smaller context |
| Multi-stage builds | Smaller final images |
| Scan images for vulnerabilities | Security compliance |

### Kubernetes Best Practices

| Practice | Benefit |
|----------|---------|
| Use pod labels for organization | Better visibility |
| Set resource limits | Prevent noisy neighbors |
| Use `readinessProbe` | Reliable service discovery |
| Pin plugin versions | Avoid breaking changes |
| Use namespaces | Environment isolation |
| Set pod retention to `never` | Clean up after builds |

---

## Key Takeaways

- Docker agents provide clean, isolated build environments
- The Docker Pipeline plugin integrates `docker.build`, `docker.withRegistry`, and `docker.image`
- The Kubernetes plugin dynamically provisions pods as build agents
- Sidecar containers (databases, browsers, mock services) run alongside build containers
- Helm deployments can be integrated into pipeline stages
- Always clean up Docker and K8s resources in `post { always }`

---

## Next Steps

→ Continue to [Chapter 12: Testing, Security & Compliance]({{< relref "12-testing-security-and-compliance" >}})
