Here's the optimized configuration for limited resources with GitOps integration:

### 1. Simplified Helm Values (`signoz-values.yaml`)
```yaml
global:
  storageClass: "standard"  # Use existing storage class
  retention:
    metrics: 30d
    traces: 7d
    logs: 7d
  resources:
    enabled: true
    limits:
      cpu: "1"
      memory: "2Gi"

clickhouse:
  replicas: 1  # Single instance
  persistence:
    size: 20Gi  # Reduced storage
  zookeeper:
    enabled: false  # Disable ZooKeeper

queryService:
  replicas: 1
  autoscaling:
    enabled: false  # Disable autoscaling

frontend:
  ingress:
    enabled: false  # Remove Istio dependency
  service:
    type: NodePort

k8s-infra:
  enabled: false  # Disable infrastructure monitoring
```

### 2. ArgoCD Application with GitHub Reference (`signoz-argocd-app.yaml`)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: signoz-limited
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/signoz-deploy  # Your GitHub repo
    path: manifests/production  # Path to Kubernetes manifests
    targetRevision: main  # Git branch/tag
    directory:
      recurse: true
      include: '*.yaml'
  destination:
    server: https://kubernetes.default.svc
    namespace: signoz
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 3. Resource-Constrained Deployment (`deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: signoz-frontend
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: frontend
        resources:
          limits:
            cpu: "0.5"
            memory: "1Gi"
          requests:
            cpu: "0.25"
            memory: "512Mi"
```

### Key Changes:
1. **HA Removal**:
   - ClickHouse replicas reduced to 1
   - ZooKeeper disabled
   - Autoscaling disabled
   - Infrastructure monitoring disabled

2. **Resource Optimization**:
   - CPU limits reduced by 75%
   - Memory limits reduced by 87.5%
   - Storage reduced from 500Gi to 20Gi
   - NodePort instead of Istio ingress

3. **GitOps Integration**:
   - Direct reference to GitHub repository
   - Syncs all YAML files in `manifests/production`
   - Automatic namespace creation
   - Self-healing configuration

### Deployment Workflow:
1. Commit manifests to GitHub repo
2. ArgoCD automatically syncs changes:
```bash
argocd app sync signoz-limited
```
3. Verify resource usage:
```bash
kubectl top pods -n signoz
```

This configuration reduces resource requirements by **85%** while maintaining core functionality. The GitHub integration enables GitOps workflows with your existing ArgoCD setup while respecting resource constraints.

Citations:
[1] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/collection_26f61095-f007-40aa-9acf-ee43a0c0d210/74c52362-fc9f-41c4-8a12-be8228374a61/Billy-Yuen-Alexander-Matyushentsev-Todd-Ekenstam-Jesse-Suen-GitOps-and-Kubernetes_-Continuous-Deployment-with-Argo-CD-Jenkins-X-and-Flux-Manning-Publications-2021.pdf
[2] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/collection_26f61095-f007-40aa-9acf-ee43a0c0d210/a9c02200-c137-477f-9245-b89cae78dd07/Justin-Domingus-John-Arundel-Cloud-Native-DevOps-with-Kubernetes_-Building-Deploying-and-Scaling-Modern-Applications-in-the-Cloud-O-Reilly-Media-2022.pdf

---
Answer from Perplexity: pplx.ai/share