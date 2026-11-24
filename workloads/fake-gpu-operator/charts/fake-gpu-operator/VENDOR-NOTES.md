# Vendored-chart divergence from ~/git/fake-gpu-operator

This is a vendored copy of the fork's chart (deploy/fake-gpu-operator) with
the kustomize/ArgoCD-specific deltas below. Re-apply them when re-vendoring:

1. gpu-operator subchart dependency REMOVED (Chart.yaml + charts/ tgz):
   unused in this environment (gpuOperator disabled), and its bundled
   nvidia.com CRDs collide with the chart's own under includeCRDs.
2. crds/resource.nvidia.com_computedomains.yaml REMOVED: the
   dra-driver-nvidia-gpu 0.5.1 subchart ships the same CRD; with both
   present kustomize fails with "already registered id
   CustomResourceDefinition.v1...computedomains".
