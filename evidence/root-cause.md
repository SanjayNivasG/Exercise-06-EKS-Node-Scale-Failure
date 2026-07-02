# Root Cause Analysis

## Incident

Application failed to scale during increased CPU utilization.

## Investigation

- HPA detected high CPU usage (99%).
- Desired replicas increased from 2 to 4.
- Scheduler attempted to place new Pods.
- Two Pods remained in Pending state.
- Cluster Autoscaler was not installed in the cluster.

## Root Cause

The Kubernetes Horizontal Pod Autoscaler worked correctly.

The scheduler attempted to schedule additional Pods, but there were insufficient CPU resources available on the worker nodes.

Since Cluster Autoscaler was not installed or configured, no additional worker nodes were provisioned automatically.

As a result, the application could not scale beyond the available cluster capacity.

