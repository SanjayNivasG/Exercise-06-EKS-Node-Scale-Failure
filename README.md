# Exercise 06 – EKS Node Scale Failure Investigation

## Project Overview

This project demonstrates a real-world Kubernetes production incident where an application could not scale during increased CPU demand. The investigation focuses on identifying whether the issue originated from the Horizontal Pod Autoscaler (HPA), Kubernetes Scheduler, worker nodes, or Cluster Autoscaler.

The exercise reproduces a production scenario in Amazon EKS where HPA successfully requests additional replicas, but new Pods remain in the **Pending** state because the cluster does not have enough CPU resources and no Cluster Autoscaler is available to provision new worker nodes.

---

## Scenario

**Incident**

Application failed to scale during increased CPU utilization.

### Symptoms

- High CPU utilization
- HPA increased desired replicas
- New Pods remained in Pending state
- Application capacity did not increase

---

## Architecture

```
                    Users
                      │
                      ▼
             Payment Application
                      │
                      ▼
          Kubernetes Deployment
                      │
                      ▼
     Horizontal Pod Autoscaler (HPA)
                      │
                      ▼
       Scheduler Creates New Pods
                      │
                      ▼
        Worker Nodes Out of CPU
                      │
                      ▼
          Pods Remain Pending
                      │
                      ▼
 Cluster Autoscaler Investigation
```

---

# Environment

| Component | Value |
|-----------|--------|
| Platform | Amazon EKS |
| Kubernetes | v1.34 |
| Metrics Server | Installed |
| Worker Nodes | 2 |
| Container Image | BusyBox (CPU Stress) |
| Namespace | payment |

---

# Objectives

- Deploy an application in Amazon EKS
- Configure Horizontal Pod Autoscaler
- Generate high CPU utilization
- Observe automatic scaling
- Investigate Pending Pods
- Identify scheduler failure
- Analyze cluster capacity
- Investigate Cluster Autoscaler configuration
- Perform root cause analysis

---

# Project Structure

```
Exercise-06-EKS-Node-Scale-Failure/
│
├── README.md
├── namespace.yaml
├── deployment.yaml
├── service.yaml
├── hpa.yaml
│
├── evidence/
│   ├── hpa.txt
│   ├── nodes.txt
│   ├── pending-pods.txt
│   ├── scheduler-events.txt
│   └── root-cause.md
│
└── screenshots/
    ├── 01-hpa-scaling.png
    ├── 02-pods-running-pending.png
    ├── 03-pending-pod-events.png
    ├── 04-node-resource-usage.png
    ├── 05-kube-system-deployments.png
    └── 06-project-structure.png
```

---

# Investigation Steps

### 1. Deploy Application

Created a Kubernetes Deployment with CPU resource requests and limits.

---

### 2. Expose Application

Created a ClusterIP Service for internal communication.

---

### 3. Configure HPA

Configured Horizontal Pod Autoscaler.

- Minimum Replicas : 2
- Maximum Replicas : 15
- Target CPU : 50%

---

### 4. Generate CPU Load

Used a CPU-intensive BusyBox container to simulate production workload.

---

### 5. Monitor HPA

Verified that HPA detected high CPU utilization and increased the desired replica count.

Example:

```
CPU Utilization

99%

Desired Replicas

4
```

---

### 6. Observe Scheduler

Scheduler attempted to place additional Pods.

Result:

```
Running Pods

2

Pending Pods

2
```

---

### 7. Investigate Pending Pods

Described Pending Pods to identify scheduling failure.

Scheduler Events:

```
FailedScheduling

Insufficient CPU
```

---

### 8. Verify Node Resources

Checked worker node CPU utilization.

Both worker nodes were consuming over 50% CPU with no available capacity for additional Pods.

---

### 9. Investigate Cluster Autoscaler

Verified Cluster Autoscaler availability.

Result:

```
Cluster Autoscaler deployment not found.
```

No automatic node provisioning was available.

---

# Root Cause Analysis

## Findings

- Horizontal Pod Autoscaler worked correctly.
- CPU utilization exceeded configured threshold.
- HPA increased desired replicas.
- Kubernetes Scheduler attempted to schedule new Pods.
- Worker nodes had insufficient CPU resources.
- New Pods entered Pending state.
- Cluster Autoscaler was not installed/configured.
- No additional worker nodes were provisioned.

---

# Root Cause

The application failed to scale because worker nodes had insufficient CPU resources.

Although the Horizontal Pod Autoscaler correctly increased the desired replica count, Kubernetes Scheduler could not schedule additional Pods due to resource exhaustion.

Since Cluster Autoscaler was unavailable, the cluster could not automatically add new worker nodes, leaving Pods in the Pending state.

---

# Solution

- Verify HPA configuration
- Monitor node CPU utilization
- Configure appropriate resource requests
- Install and configure Cluster Autoscaler
- Use managed node groups for automatic scaling
- Monitor cluster capacity using Prometheus and Grafana

---

# Commands Used

```bash
kubectl get deployment -n payment

kubectl get pods -n payment

kubectl get hpa -n payment

kubectl describe hpa payment-hpa -n payment

kubectl describe pod <pending-pod> -n payment

kubectl top pods -n payment

kubectl top nodes

kubectl get deployment -n kube-system
```

---

# Key Learning Outcomes

- Kubernetes Deployment
- Resource Requests and Limits
- Horizontal Pod Autoscaler
- Metrics Server
- Kubernetes Scheduler
- Pod Scheduling
- Pending Pods Investigation
- CPU Resource Management
- Cluster Capacity Planning
- Production Troubleshooting
- Root Cause Analysis
- Amazon EKS Operations

---

# Screenshots

- HPA Scaling
- Running and Pending Pods
- Scheduler Events
- Node Resource Usage
- Cluster Autoscaler Verification
- Project Structure

---

# Skills Demonstrated

- Amazon EKS
- Kubernetes
- Horizontal Pod Autoscaler
- Kubernetes Scheduler
- Cluster Capacity Analysis
- Production Incident Investigation
- Root Cause Analysis
- DevOps Troubleshooting
- YAML Configuration
- Linux CLI

---

## Author

**Sanjay Nivas G**
