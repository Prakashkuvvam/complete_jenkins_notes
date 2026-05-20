---
title: "Ch 14: Production-Grade Jenkins"
weight: 14
---

# Production-Grade Jenkins

## Learning Objectives

After reading this chapter, you will be able to:
- Design a highly available Jenkins architecture
- Implement backup and disaster recovery strategies
- Monitor Jenkins with alerts and dashboards
- Optimize Jenkins performance at scale
- Manage plugin upgrades safely
- Plan capacity and scaling

---

## 14.1 High Availability Architecture

### Active/Passive with Shared Storage

```
┌─────────────────┐     ┌─────────────────┐
│   Jenkins       │     │   Jenkins       │
│   Active        │     │   Passive       │
│   (Primary)     │     │   (Standby)     │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └──────────┬────────────┘
                    │
         ┌──────────▼────────────┐
         │    Shared NFS/EFS     │
         │   /var/jenkins_home   │
         └───────────────────────┘
                    │
         ┌──────────▼────────────┐
         │     Load Balancer     │
         │  (HAProxy/Nginx/ALB)  │
         └───────────────────────┘
                    │
                    ▼
                Users (jenkins.example.com)
```

### Key Components

| Component | Recommendation |
|-----------|---------------|
| **Load balancer** | Nginx, HAProxy, or AWS ALB |
| **Shared storage** | AWS EFS, NFS, or GlusterFS |
| **Database** | External PostgreSQL (for modern Jenkins) |
| **Agents** | Kubernetes or EC2 spot instances |
| **Backups** | Automated to S3 or equivalent |

---

## 14.2 Docker-Compose with High Availability

```yaml
version: '3.8'
services:
  lb:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    
  jenkins-1:
    image: jenkins/jenkins:lts
    volumes:
      - jenkins_home:/var/jenkins_home
    environment:
      - CASC_JENKINS_CONFIG=/var/jenkins_home/casc_configs/jenkins.yaml
    networks:
      - jenkins
  
  jenkins-2:
    image: jenkins/jenkins:lts
    volumes:
      - jenkins_home:/var/jenkins_home
    environment:
      - CASC_JENKINS_CONFIG=/var/jenkins_home/casc_configs/jenkins.yaml
    networks:
      - jenkins

volumes:
  jenkins_home:
    driver: rclone  # Shared volume

networks:
  jenkins:
```

---

## 14.3 Backup Strategy

### Full Backup Script

```bash
#!/bin/bash
# Full backup of Jenkins
JENKINS_HOME="/var/jenkins_home"
BACKUP_BUCKET="s3://jenkins-backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/tmp/jenkins-backup-$DATE"

mkdir -p "$BACKUP_DIR"

# Stop Jenkins to ensure consistency
systemctl stop jenkins
sleep 5

# Backup configuration (excluding large directories)
rsync -av --delete \
    --exclude='jobs/*/builds/*' \
    --exclude='workspace/*' \
    --exclude='logs/*' \
    --exclude='caches/*' \
    "$JENKINS_HOME/" "$BACKUP_DIR/config/"

# Export job configurations
find "$JENKINS_HOME/jobs" -name "config.xml" -exec cp --parents {} "$BACKUP_DIR/" \;

# Backup credentials
cp "$JENKINS_HOME/secrets/"* "$BACKUP_DIR/secrets/" 2>/dev/null

# Dump database (if using external DB)
pg_dump -h db-host -U jenkins jenkinsdb > "$BACKUP_DIR/jenkinsdb.sql"

# Start Jenkins again
systemctl start jenkins

# Compress and upload
cd /tmp
tar czf "jenkins-backup-$DATE.tar.gz" "jenkins-backup-$DATE"
aws s3 cp "jenkins-backup-$DATE.tar.gz" "$BACKUP_BUCKET/"

# Cleanup
rm -rf "$BACKUP_DIR" "jenkins-backup-$DATE.tar.gz"

echo "Backup complete: $BACKUP_BUCKET/jenkins-backup-$DATE.tar.gz"
```

### Backup Schedule

| Frequency | What to Backup | Retention |
|-----------|---------------|-----------|
| **Hourly** | Job config changes (rsync) | 24 hours |
| **Daily** | Full backup of config + jobs | 30 days |
| **Weekly** | Full backup + builds archive | 3 months |
| **Monthly** | Full backup to cold storage | 12 months |

---

## 14.4 Monitoring

### Prometheus & Grafana

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['jenkins:8080']
```

### Key Metrics to Monitor

| Metric | Warning | Critical | Action |
|--------|---------|----------|--------|
| Queue length | > 10 | > 50 | Add more agents |
| Executor count | > 80% | > 95% | Scale agents |
| Response time | > 2s | > 5s | Check resources |
| Disk usage | > 80% | > 90% | Clean/expand |
| Memory usage | > 75% | > 90% | Increase heap |
| Agent offline | > 1 | > 5 | Investigate |

### Health Check Endpoint

```groovy
// Custom health check script
import jenkins.model.*

def instance = Jenkins.instance
def queue = instance.queue
def computers = instance.computers

println "Health Check Report"
println "==================="
println "Status: ${instance.isQuietingDown() ? 'Quieting' : 'Running'}"
println "Executors: ${computers*.countBusy.sum()}/${computers*.countExecutor.sum()}"
println "Queue length: ${queue.items.size()}"
println "Uptime: ${System.currentTimeMillis() - instance.toComputer().connectTime}"
println "Memory: ${Runtime.runtime.totalMemory() - Runtime.runtime.freeMemory()} / ${Runtime.runtime.maxMemory()}"
```

---

## 14.5 Performance Tuning

### JVM Tuning

```bash
# JAVA_OPTS for production Jenkins
export JAVA_OPTS="-Xms8g -Xmx8g \
  -XX:MaxRAMPercentage=75.0 \
  -XX:+UseG1GC \
  -XX:+ParallelRefProcEnabled \
  -XX:+DisableExplicitGC \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/jenkins/heapdump.hprof \
  -Djenkins.install.runSetupWizard=false"
```

### Jenkins System Configuration

| Setting | Recommendation | Reason |
|---------|---------------|--------|
| Executors on master | 0 | Master should only coordinate |
| SCM checkout retry | 3 | Handle transient network issues |
| Quiet period | 5 seconds | Batch multiple SCM changes |
| Build discarder | Keep 30 days / 100 builds | Prevent disk bloat |
| Workspace cleanup | After each build | Free disk space |

### Plugin Optimization

1. **Remove unused plugins** — Every plugin adds overhead
2. **Pin plugin versions** — Avoid unexpected upgrades
3. **Use LTS Jenkins** — Stable releases with backported fixes
4. **Schedule plugin upgrades** — During maintenance windows
5. **Cache plugins** — Use a custom image with pre-installed plugins

---

## 14.6 Disaster Recovery

### Recovery Procedure

```bash
#!/bin/bash
# Disaster recovery script
BACKUP_FILE="s3://jenkins-backups/jenkins-backup-latest.tar.gz"
JENKINS_HOME="/var/jenkins_home"

# 1. Provision new server
echo "Step 1: Provision server..."
# (Use Terraform/CloudFormation to provision)

# 2. Install dependencies
echo "Step 2: Install Java, Docker, etc..."
apt update && apt install -y openjdk-17-jre docker.io

# 3. Restore backup
echo "Step 3: Restore configuration..."
cd /tmp
aws s3 cp "$BACKUP_FILE" .
tar xzf "$(basename $BACKUP_FILE)"
rsync -av jenkins-backup-*/ $(JENKINS_HOME)

# 4. Start Jenkins
echo "Step 4: Start Jenkins..."
docker run -d \
  -p 8080:8080 \
  -v $JENKINS_HOME:/var/jenkins_home \
  jenkins/jenkins:lts

# 5. Verify
echo "Step 5: Verify recovery..."
curl -f http://localhost:8080/login
echo "Recovery complete!"
```

### RTO & RPO Targets

| Metric | Target |
|--------|--------|
| **Recovery Time Objective (RTO)** | < 1 hour |
| **Recovery Point Objective (RPO)** | < 15 minutes |
| **Max acceptable data loss** | Last 15 minutes of job history |

---

## 14.7 Upgrade Strategy

### Blue/Green Upgrade

```yaml
version: '3.8'
services:
  jenkins-blue:
    image: jenkins/jenkins:2.440.1  # Current version
    volumes:
      - jenkins_home:/var/jenkins_home
  
  jenkins-green:
    image: jenkins/jenkins:2.440.2  # New version
    volumes:
      - jenkins_home-new:/var/jenkins_home
    profiles:
      - staging
```

### Upgrade Checklist

- [ ] Read release notes for breaking changes
- [ ] Check plugin compatibility with new version
- [ ] Test upgrade in staging environment
- [ ] Take a full backup
- [ ] Schedule maintenance window
- [ ] Upgrade Jenkins
- [ ] Upgrade plugins
- [ ] Verify all pipeline jobs run correctly
- [ ] Monitor for 24 hours post-upgrade

---

## Key Takeaways

- High availability requires shared storage, load balancers, and external database
- Automate backups with scripts — store off-server (S3, S3-compatible storage)
- Monitor queue length, executor usage, and response times
- Tune JVM with G1GC and appropriate heap size
- Test disaster recovery procedures regularly
- Plan upgrades with blue/green deployments in staging

---

## Next Steps

→ Continue to [Chapter 15: Exam Preparation Guide]({{< relref "15-exam-preparation-guide" >}})
