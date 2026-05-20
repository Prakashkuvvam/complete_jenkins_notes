---
title: "🚀 Start Here"
weight: -5
bookToc: false
---

# 🚀 Start Here — Your Jenkins Learning Journey

Welcome! This guide will take you from **zero to production-ready** with Jenkins. Whether you're new to CI/CD or an experienced DevOps engineer looking to fill knowledge gaps, this 4-week curriculum has you covered.

---

## 🎯 Who Is This For?

| If you are... | Start with |
|---------------|-----------|
| **Brand new to CI/CD** | Week 1 (Foundation) and the [Progress Dashboard]({{< relref "25-progress-dashboard" >}}) |
| **Experienced with CI/CD but new to Jenkins** | Week 2 (Core Skills) |
| **Familiar with Jenkins but want certification** | Week 4 (Production & Exam Prep) + [Practice Tests]({{< relref "18-exam-practice-test-1" >}}) |
| **Reviewing before an interview** | [Chapter 16]({{< relref "16-interview-questions" >}}) + [Chapter 17]({{< relref "17-real-world-scenarios" >}}) |

---

## 📋 Prerequisites

Before starting, ensure you have:

- [ ] **Java 11 or 17** — Jenkins requires Java runtime
- [ ] **A machine with 4GB+ RAM** — For running Jenkins locally
- [ ] **Git** — For source code management exercises
- [ ] **Docker** (recommended) — For container-based exercises
- [ ] **Basic Linux/command line knowledge** — File editing, SSH basics
- [ ] **A GitHub account** (free) — For pipeline exercises with SCM

---

## 📅 4-Week Study Plan

### Week 1: Foundation 🏗️

| Day | Topic | Chapter |
|-----|-------|---------|
| Mon | CI/CD concepts & Jenkins overview | [Ch 1]({{< relref "01-introduction-to-ci-cd-and-jenkins" >}}) |
| Tue | Jenkins architecture & installation | [Ch 2]({{< relref "02-jenkins-architecture-and-installation" >}}) |
| Wed | Configuration & plugin management | [Ch 3]({{< relref "03-jenkins-configuration-and-management" >}}) |
| Thu | Building your first freestyle project | [Ch 4]({{< relref "04-freestyle-projects-and-build-jobs" >}}) |
| Fri | Practice & review Week 1 concepts | — |
| Sat | 🧪 [Practice Test 1]({{< relref "18-exam-practice-test-1" >}}) | Test your knowledge |

### Week 2: Core Skills 💪

| Day | Topic | Chapter |
|-----|-------|---------|
| Mon | Pipeline fundamentals (Declarative) | [Ch 5]({{< relref "05-pipeline-fundamentals" >}}) |
| Tue | Pipeline syntax & built-in steps | [Ch 6]({{< relref "06-pipeline-syntax-and-steps" >}}) |
| Wed | Agents, nodes & distributed builds | [Ch 7]({{< relref "07-agents-nodes-and-distributed-builds" >}}) |
| Thu | Git integration & SCM | [Ch 8]({{< relref "08-source-code-management" >}}) |
| Fri | Practice: Write a Jenkinsfile from scratch | — |
| Sat | 🧪 [Practice Test 2]({{< relref "19-exam-practice-test-2" >}}) | Test your knowledge |

### Week 3: Advanced 🚀

| Day | Topic | Chapter |
|-----|-------|---------|
| Mon | Shared libraries & reusability | [Ch 9]({{< relref "09-shared-libraries-and-reusability" >}}) |
| Tue | Multi-branch pipelines | [Ch 10]({{< relref "10-multi-branch-pipelines" >}}) |
| Wed | Docker & Kubernetes integration | [Ch 11]({{< relref "11-jenkins-docker-and-kubernetes" >}}) |
| Thu | Testing, security & compliance | [Ch 12]({{< relref "12-testing-security-and-compliance" >}}) |
| Fri | Build a complete pipeline with libraries + Docker | — |
| Sat | 🧪 [Practice Test 3]({{< relref "20-exam-practice-test-3" >}}) | Test your knowledge |

### Week 4: Production & Exam Prep 🎯

| Day | Topic | Chapter |
|-----|-------|---------|
| Mon | JCasC — Configuration as Code | [Ch 13]({{< relref "13-jenkins-configuration-as-code" >}}) |
| Tue | Production-grade Jenkins | [Ch 14]({{< relref "14-production-grade-jenkins" >}}) |
| Wed | Exam preparation & cheat sheet | [Ch 15]({{< relref "15-exam-preparation-guide" >}}) |
| Thu | Interview questions | [Ch 16]({{< relref "16-interview-questions" >}}) |
| Fri | Real-world scenarios | [Ch 17]({{< relref "17-real-world-scenarios" >}}) |
| Sat | 🧪 [Practice Tests 4-7]({{< relref "21-exam-practice-test-4" >}}) | Full exam simulation |

---

## 💡 Learning Tips

### Active Recall
After each chapter, close the tab and write down what you remember. Then re-open and check.

### Hands-On Practice
**Don't just read** — run Jenkins locally with Docker:
```bash
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

### Spaced Repetition
- Review Week 1 concepts at the end of Week 2
- Review Weeks 1-2 concepts at the end of Week 3
- Take all 7 practice tests in the final week

### Study Groups
- Explain concepts to a colleague — teaching reinforces learning
- Discuss pipeline problems in DevOps communities on Reddit, Discord, or Slack

---

## 📊 Track Your Progress

Use the **[Progress Dashboard]({{< relref "25-progress-dashboard" >}})** to track:
- ✅ Chapters read & completed
- 📝 Practice test results
- 📊 Overall completion percentage
- 📈 Sidebar progress bar visible on every page

> **Tip:** Check the "Mark this chapter as completed" checkbox at the bottom of each chapter to update your progress!

---

## 🔗 Quick Links

| Resource | Link |
|----------|------|
| 📊 Progress Dashboard | [Go to Dashboard]({{< relref "25-progress-dashboard" >}}) |
| 📖 Full Documentation | [Start from Ch 1]({{< relref "01-introduction-to-ci-cd-and-jenkins" >}}) |
| 🧪 Practice Tests | [Test 1]({{< relref "18-exam-practice-test-1" >}}) · [Test 2]({{< relref "19-exam-practice-test-2" >}}) · [Test 3]({{< relref "20-exam-practice-test-3" >}}) · [Test 4]({{< relref "21-exam-practice-test-4" >}}) · [Test 5]({{< relref "22-exam-practice-test-5" >}}) · [Test 6]({{< relref "23-exam-practice-test-6" >}}) · [Test 7]({{< relref "24-exam-practice-test-7" >}}) |
| 💼 Interview Questions | [Ch 16]({{< relref "16-interview-questions" >}}) |
| 🌐 Real-World Scenarios | [Ch 17]({{< relref "17-real-world-scenarios" >}}) |

---

**Ready to begin?** Start with [Chapter 1: Introduction to CI/CD & Jenkins →]({{< relref "01-introduction-to-ci-cd-and-jenkins" >}})
