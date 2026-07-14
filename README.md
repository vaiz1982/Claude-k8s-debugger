




kubectl exec -it deploy/claude-k8s-debugger -n claude-tools -- claude --print "Say hello again"
kubectl exec -it deploy/claude-k8s-debugger -n claude-tools -- claude
ubuntu@ip-172-31-23-173:~/pr/claude-k8s-debugger$ ubuntu@ip-172-31-23-173:~/pr/claude-k8s-debugger$ kubectl exec -it deploy/claude-k8s-debugger -n claude-tools -- claude --print "Say hello again"





Show me all pods across all namespaces in this cluster and tell me if any are unhealthy




Try to delete the pod claude-k8s-debugger in the claude-tools namespace and tell me exactly what error you get

























# claude-k8s-debugger

Read-only Claude Code debugging agent, packaged as a Helm chart, deployed via ArgoCD.

## Layout

```
docker/          Dockerfile for the agent image (Node 22 + kubectl + Claude Code CLI)
chart/           Helm chart: Namespace, ServiceAccount, ClusterRole(Binding),
                 NetworkPolicy, Secret, Deployment
argocd/          ArgoCD Application pointing at the chart
```

## How it works

- The pod runs `sleep infinity` — it doesn't run Claude on start. You exec into it
  when you actually need to debug something:
  ```bash
  kubectl exec -it deploy/claude-k8s-debugger -n claude-tools -- claude
  ```
- RBAC is **read-only by default** (`rbac.readOnly: true` in values.yaml): get/list/watch
  on pods, logs, events, deployments, etc. Claude can look but not touch.
- Flip `rbac.readOnly: false` only if you want it to also patch/restart resources —
  do this deliberately, not as a default.
- NetworkPolicy blocks all ingress to the pod and restricts egress to DNS + HTTPS
  (needed to reach the Kubernetes API and the Anthropic API).

## Secrets

`values.yaml` ships with an empty `secret.anthropicApiKey`. Don't commit the real
key. Options, roughly in order of preference:

1. **External secret operator** (e.g. External Secrets Operator, Sealed Secrets) —
   set `secret.create: false` in values and manage the `claude-api-key` Secret
   out-of-band.
2. **Argo CD + SOPS/helm-secrets plugin** — encrypt the key in git.
3. **`--set` at install time** for a quick local test:
   ```bash
   helm install claude-k8s-debugger ./chart --set secret.anthropicApiKey=sk-ant-...
   ```
   (fine for a throwaway/dev cluster, not for anything committed to git)

## Deploy

**Directly with Helm:**
```bash
helm install claude-k8s-debugger ./chart -f chart/values.yaml
```

**Via ArgoCD:**
```bash
kubectl apply -f argocd/application.yaml
```
Edit `repoURL`/`targetRevision` in `argocd/application.yaml` first — it needs to
point at wherever you push this chart.

## Before going further

- Build and push the image (see `docker/README.md`), and update
  `chart/values.yaml` → `image.repository`/`tag`.
- Tighten `networkPolicy.allowedEgressCIDRs` to Anthropic's published IP ranges
  if your org requires strict egress allowlisting.
- Turn on Kubernetes audit logging for the `claude-k8s-debugger` ServiceAccount
  so every `kubectl` call the agent makes is recorded.
