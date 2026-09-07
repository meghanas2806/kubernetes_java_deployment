# HPA Scaling Experiment Evidence

This directory contains exported evidence from the Kubernetes Horizontal Pod Autoscaler (HPA) experiment for the `shopfront` service.

## Experiment objective

Demonstrate that Kubernetes automatically scales the `shopfront` Deployment based on CPU utilization and scales the workload back down when the load is removed.

## HPA configuration

| Setting | Value |
|---|---:|
| Target workload | `shopfront` Deployment |
| Minimum replicas | 1 |
| Maximum replicas | 3 |
| CPU target | 60% |
| CPU request per pod | 100m |
| CPU limit per pod | 500m |
| Scale-up stabilization | 0 seconds |
| Scale-down stabilization | 60 seconds |

The HPA CPU utilization target is calculated relative to the pod's CPU **request**, not its CPU limit.

With a CPU request of `100m`, a 60% HPA target corresponds to approximately `60m` CPU utilization per pod.

## Observed scaling

Prometheus historical data captured the following replica transitions during the experiment:

| Time (UTC) | Available replicas |
|---|---:|
| 20:38:30 → 21:22:30 | 1 |
| 21:22:30 → 21:27:00 | 2 |
| 21:27:00 → 21:37:30 | 3 |
| 21:37:30 → 22:00:00 | 1 |

### Observed pattern

**1 → 2 → 3 → 1 replicas**

The workload was increased using a temporary load-generator pod. CPU utilization increased above the HPA target, causing the `shopfront` Deployment to scale out. After the load was stopped, CPU utilization decreased and the HPA eventually returned the Deployment to its minimum of one replica.

## Evidence files

### `hpa-current.txt`

Current HPA status after the experiment.

### `hpa-config.yaml`

Export of the Kubernetes HPA configuration, including:

- minimum and maximum replicas
- CPU target
- scaling behavior
- current metrics
- HPA conditions

### `shopfront_hpa_history.json`

Raw Prometheus `query_range` export containing the historical `shopfront` available-replica metric.

### `shopfront_replica_history.csv`

CSV representation of the historical replica data for easier analysis.

### `shopfront_cpu_hpa_history.json`

Raw Prometheus historical CPU utilization data used to correlate CPU load with HPA scaling.

### `hpa-scaling-summary.txt`

Human-readable summary of the observed scaling transitions.

### `deployments-current.txt`

Current state of the application Deployments after the experiment.

### `pods-current.txt`

Current state of the application Pods after the experiment.

## What this experiment demonstrates

This experiment provides evidence of several Kubernetes capabilities:

- Horizontal Pod Autoscaling based on CPU utilization
- Kubernetes reconciliation of desired and actual replica counts
- Automatic scale-out under increased workload
- Automatic scale-in after workload reduction
- Resource requests used as the basis for HPA CPU utilization
- Prometheus collection of historical Kubernetes metrics
- Grafana visualization of Kubernetes resource utilization

## Important observation

CPU utilization values above 100% in the exported Prometheus HPA history do not mean that a container exceeded its configured CPU limit.

The HPA utilization calculation is based on CPU usage relative to the configured CPU request. Since the `shopfront` container has a request of `100m` and a limit of `500m`, usage can exceed 100% of the request while remaining below the CPU limit.

## Monitoring

The Kubernetes metrics pipeline used for this experiment was:

Java services  
→ Kubernetes Pods  
→ Kubelet / cAdvisor + kube-state-metrics  
→ Prometheus  
→ Grafana

Grafana was used to visualize CPU, memory, network, and other Kubernetes resource metrics for the `shopping-app` namespace.

## Reproducibility

The experiment can be reproduced by:

1. Deploying the application to the `shopping-app` namespace.
2. Ensuring Metrics Server is available.
3. Applying the `shopfront` HPA.
4. Generating sustained HTTP traffic against the `shopfront` service.
5. Observing CPU utilization and HPA replica changes.
6. Stopping the load generator.
7. Observing the subsequent scale-down.

The exported Prometheus data in this directory preserves the results of the completed experiment independently of the current live cluster state.
