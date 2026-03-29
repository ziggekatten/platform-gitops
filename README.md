# platform-gitops

GitOps repo for cluster-scoped components and Argo CD bootstrap.

## What This Repo Manages

- Argo CD bootstrap and self-management
- Argo CD `AppProject` definitions
- cluster root applications for `dev`, `test`, `prod`, and `clients`
- platform `ApplicationSet` entrypoints per cluster
- namespaces
- Gatekeeper
- Gateway API resources
- `CiliumNetworkPolicy` resources
- shared policy placeholders such as quotas and limits

## What This Repo Does Not Manage

Cilium is assumed to already be installed in the clusters.

Argo CD does not install or own the Cilium deployment itself in this setup.

The repo does contain manual Cilium manifests that you can apply separately when you want them tracked in Git, for example:

- `platform/cilium/base/bgp-policies.yaml`

That file is useful for advertising Gateway-created `LoadBalancer` IPs over BGP without putting the Cilium installation under Argo CD.

## Current Dev Setup

The current `dev` environment is wired like this:

- `GatewayClass`: `cilium`
- Gateway dataplane pods: `app.kubernetes.io/name=cilium-envoy` in `kube-system`
- dedicated Argo CD Gateway: `argocd/argocd`
- Argo CD hostname: `argocd.dev.bogus.net`
- Argo CD exposed over plain HTTP

## Bootstrap Flow

Use this flow for a new cluster.

1. Install Argo CD

```powershell
kubectl --kubeconfig .\kubeconfig apply --server-side --force-conflicts -k .\platform-gitops\bootstrap\argocd\install\overlays\<env>
```

2. Wait for Argo CD to become ready

```powershell
kubectl --kubeconfig .\kubeconfig get crd | Select-String argoproj.io
kubectl --kubeconfig .\kubeconfig -n argocd get pods
```

3. Apply Argo CD projects

```powershell
kubectl --kubeconfig .\kubeconfig apply -k .\platform-gitops\bootstrap\argocd\projects
```

4. Apply the self-managing Argo CD application

```powershell
kubectl --kubeconfig .\kubeconfig apply -f .\platform-gitops\bootstrap\argocd\apps\argocd-<env>.yaml
```

5. Apply the cluster root application

```powershell
kubectl --kubeconfig .\kubeconfig apply -f .\platform-gitops\bootstrap\argocd\root-apps\<env>-root.yaml
```

6. Verify that Argo CD created the expected applications

```powershell
kubectl --kubeconfig .\kubeconfig -n argocd get applications
kubectl --kubeconfig .\kubeconfig -n argocd get applicationsets
```

## Dev Access

The `dev` Argo CD UI is exposed at:

- `http://argocd.dev.bogus.net`

If you are using a Windows `hosts` file in the lab, add:

```text
192.168.3.11 argocd.dev.bogus.net
```

Then flush DNS:

```powershell
ipconfig /flushdns
```

## Argo CD Initial Login

Default username:

```text
admin
```

Get the initial admin password:

```powershell
kubectl --kubeconfig .\kubeconfig -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_)) }
```

After logging in, change the admin password.

## Manual Cilium BGP Advertisement Update

If Gateway-created `LoadBalancer` services are not being announced over BGP, apply:

```powershell
kubectl --kubeconfig .\kubeconfig apply -f .\platform-gitops\platform\cilium\base\bgp-policies.yaml
```

This updates the `CiliumBGPAdvertisement` so Gateway-backed services such as Argo CD can be advertised.
