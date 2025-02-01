# ADR 0001: Adopt GitOps with ArgoCD for Signoz Deployment

## Status
Accepted

## Context
- Signoz is used for monitoring, tracing, and logging, but our resource environment is limited.
- High availability (HA) components (e.g., multiple replicas, ZooKeeper) add overhead that is unnecessary in our constrained environment.
- GitOps practices streamline deployments, enabling self-healing and automated sync without manual intervention.

## Decision
- **GitOps Integration:** We chose ArgoCD to automatically manage and deploy Kubernetes manifests stored in our GitHub repository.
- **Simplified Deployment:** We configure Signoz with reduced resource allocations (CPU, memory, storage) and disable HA components.
- **Automated Sync:** ArgoCD is configured for automated sync with self-heal and namespace management.

## Consequences
- **Pros:**  
  - Reduced resource consumption.
  - Automated, self-healing deployments via ArgoCD.
  - Simplified operational management and version control through GitOps.
  
- **Cons:**  
  - Limited fault tolerance given the reduced replicas and resources.
  - May require adjustments as resource constraints or usage patterns change.

## References
- [GitOps and ArgoCD Documentation](https://argoproj.github.io/argo-cd/)
- [Signoz Documentation](https://signoz.io/) 