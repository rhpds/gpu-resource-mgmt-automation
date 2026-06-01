# gpu-resource-mgmt-automation

GitOps repository for **GPUaaS using Kueue with Fake GPUs** — managed by Argo CD via the App of Apps pattern.

## Structure

```
.
├── bootstrap/                        # Root App of Apps — ArgoCD watches this path
│   ├── kustomization.yaml            # Lists all Application YAMLs
│   └── example-workload-app.yaml    # Example: deploys workloads/example-workload/
└── workloads/
    └── example-workload/            # Manifests for the example workload
        ├── kustomization.yaml
        ├── namespace.yaml
        └── deployment.yaml
```

## How it works

1. The AgnosticD `ocp4_workload_gitops_bootstrap` role creates a root Argo CD `Application` pointing at `bootstrap/` in this repo.
2. Argo CD applies everything in `bootstrap/` — each YAML there is itself an `Application` pointing at a subdirectory under `workloads/`.
3. Adding a new component = create `workloads/<name>/` with your manifests + add a new `Application` YAML in `bootstrap/` + register it in `bootstrap/kustomization.yaml`.

## Adding a new application

1. Create your manifests under `workloads/<your-app>/`:
   ```
   workloads/your-app/
   ├── kustomization.yaml
   ├── namespace.yaml
   └── <your manifests>
   ```

2. Create `bootstrap/your-app-app.yaml`:
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: your-app
     namespace: openshift-gitops
   spec:
     project: default
     source:
       repoURL: https://github.com/rhpds/gpu-resource-mgmt-automation
       targetRevision: main
       path: workloads/your-app
     destination:
       server: https://kubernetes.default.svc
       namespace: your-app
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
   ```

3. Add it to `bootstrap/kustomization.yaml`:
   ```yaml
   resources:
   - example-workload-app.yaml
   - your-app-app.yaml    # <-- add this line
   ```

4. Commit and push — Argo CD picks it up automatically.
