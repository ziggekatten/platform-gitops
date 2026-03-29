# platform-gitops

Starter GitOps repo for cluster-scoped components and Argo CD bootstrap.

This starter includes:

- Argo CD AppProjects
- pinned Argo CD HA install at `v3.3.0`
- per-cluster Argo CD install overlays for `dev`, `test`, `prod`, and `clients`
- self-managing Argo CD application manifests
- root applications for `dev`, `test`, `prod`, and `clients`
- cluster entrypoints for platform components and app workloads
- basic placeholders for namespaces, Gatekeeper, Gateway API, CiliumNetworkPolicy resources, and policies

Note: Cilium is assumed to already exist in the clusters and is not managed by Argo CD in this starter.
Argo CD manages Gateway API resources and `CiliumNetworkPolicy` objects only.
The repo also contains a manual Cilium BGP advertisement manifest in `platform/cilium/base/bgp-policies.yaml` so Gateway-created `LoadBalancer` services can be advertised consistently from Git without putting the Cilium installation itself under Argo CD.

The current Gateway starter is wired for the Cilium Gateway controller discovered in the cluster:

- `GatewayClass`: `cilium`
- Gateway dataplane pods: `app.kubernetes.io/name=cilium-envoy` in `kube-system`
- dedicated Argo CD Gateway: `argocd/argocd`
- Argo CD route hostname: `argocd.dev.bogus.net`

Bootstrap flow for a cluster:

1. `kubectl apply --server-side -k platform-gitops/bootstrap/argocd/install/overlays/<env>`
2. Wait for Argo CD CRDs and pods to be ready
3. `kubectl apply -k platform-gitops/bootstrap/argocd/projects`
4. `kubectl apply -f platform-gitops/bootstrap/argocd/apps/argocd-<env>.yaml`
5. `kubectl apply -f platform-gitops/bootstrap/argocd/root-apps/<env>-root.yaml`

Replace the placeholder Git repo URLs and Argo CD external URLs before applying.
