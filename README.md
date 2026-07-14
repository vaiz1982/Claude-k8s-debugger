next time add mcp server! 

<img width="651" height="405" alt="Screenshot 2026-07-14 at 02 10 43" src="https://github.com/user-attachments/assets/88dcd274-35cf-46e8-8f5e-c5ade4b2fde2" />

next time also if i give command to safe some on READMe.md 

<img width="737" height="673" alt="Screenshot 2026-07-14 at 02 12 07" src="https://github.com/user-attachments/assets/faf79ae0-3329-439d-926d-f916b1b300e3" />






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







<img width="711" height="802" alt="Screenshot 2026-07-14 at 01 08 09" src="https://github.com/user-attachments/assets/a53eb414-9fb4-43e3-8b29-6e2ad2fd6106" />






<img width="1472" height="721" alt="Screenshot 2026-07-14 at 01 15 32" src="https://github.com/user-attachments/assets/7f8c262d-8715-45b2-86a3-f85d6d98ec97" />






<img width="977" height="615" alt="Screenshot 2026-07-14 at 01 16 12" src="https://github.com/user-attachments/assets/bdef762e-7624-4790-b277-51797d3539dd" />






<img width="1454" height="592" alt="Screenshot 2026-07-14 at 01 16 33" src="https://github.com/user-attachments/assets/ef5fd6e2-dbb5-41bb-8d14-f12025b375e1" />








<img width="1423" height="578" alt="Screenshot 2026-07-14 at 01 17 03" src="https://github.com/user-attachments/assets/4b6ef3bb-0de7-45e9-b15e-b4d856eb0335" />







<img width="714" height="790" alt="Screenshot 2026-07-14 at 01 17 24" src="https://github.com/user-attachments/assets/33402e4d-3e7b-4ae5-9692-5c2cf5f8b6c8" />







<img width="703" height="350" alt="Screenshot 2026-07-14 at 01 17 35" src="https://github.com/user-attachments/assets/d05679a7-9a79-4185-8815-36fc88ff4c21" />







<img width="1464" height="378" alt="Screenshot 2026-07-14 at 01 17 51" src="https://github.com/user-attachments/assets/7b3e710a-e952-45d9-b017-0ec64513b41b" />







<img width="1480" height="398" alt="Screenshot 2026-07-14 at 01 17 58" src="https://github.com/user-attachments/assets/62645a86-c208-4fa2-be00-1180b51bdd3e" />







<img width="1106" height="602" alt="Screenshot 2026-07-14 at 01 18 25" src="https://github.com/user-attachments/assets/e70e3863-c7bf-4ba5-8dd1-eb190758b33c" />






<img width="1202" height="594" alt="Screenshot 2026-07-14 at 01 18 38" src="https://github.com/user-attachments/assets/725a480e-bf36-471d-b02c-2f91a58b8c8a" />








<img width="866" height="594" alt="Screenshot 2026-07-14 at 01 18 51" src="https://github.com/user-attachments/assets/21c62696-e589-4385-8aa9-23192a13e98a" />








<img width="1471" height="582" alt="Screenshot 2026-07-14 at 01 19 24" src="https://github.com/user-attachments/assets/13e12df4-c72b-4147-b4bf-19c8e496596f" />








<img width="662" height="549" alt="Screenshot 2026-07-14 at 01 20 38" src="https://github.com/user-attachments/assets/7615aee7-aa99-4699-b082-35ca166b485e" />








<img width="1306" height="585" alt="Screenshot 2026-07-14 at 01 21 01" src="https://github.com/user-attachments/assets/60a45de9-b0b3-4735-801c-52d5447d3cbe" />








<img width="1463" height="592" alt="Screenshot 2026-07-14 at 01 21 56" src="https://github.com/user-attachments/assets/d3c569ad-a567-4ee3-8cf5-18df4d110aa0" />








<img width="658" height="549" alt="Screenshot 2026-07-14 at 01 23 30" src="https://github.com/user-attachments/assets/b9923bb3-2609-4717-9d14-02e5cf905857" />









<img width="728" height="121" alt="Screenshot 2026-07-14 at 01 24 58" src="https://github.com/user-attachments/assets/aa0ff059-ecf9-4914-9bbc-e3d62f38faaa" />









<img width="642" height="451" alt="Screenshot 2026-07-14 at 01 30 10" src="https://github.com/user-attachments/assets/3b700952-ee84-476b-ac28-23efc06623a5" />










<img width="739" height="174" alt="Screenshot 2026-07-14 at 01 58 14" src="https://github.com/user-attachments/assets/5002331f-a854-419d-983c-fac376bfc784" />









<img width="661" height="660" alt="Screenshot 2026-07-14 at 02 01 21" src="https://github.com/user-attachments/assets/b9080b25-3a6c-4efb-8f57-fd210064b9b4" />









<img width="1453" height="582" alt="Screenshot 2026-07-14 at 02 03 59" src="https://github.com/user-attachments/assets/1009b78e-78f2-4081-acef-e362529adfca" />









<img width="708" height="704" alt="Screenshot 2026-07-14 at 02 05 32" src="https://github.com/user-attachments/assets/6577305e-26e1-4edc-854c-eaddd7d9cffd" />







<img width="1450" height="597" alt="Screenshot 2026-07-14 at 02 25 22" src="https://github.com/user-attachments/assets/a4e3015e-b31c-469b-883a-7d4f803c4e1c" />








<img width="1294" height="594" alt="Screenshot 2026-07-14 at 02 25 42" src="https://github.com/user-attachments/assets/db087a4a-4a99-4798-9395-b53d74b99f0b" />








<img width="1069" height="600" alt="Screenshot 2026-07-14 at 02 25 51" src="https://github.com/user-attachments/assets/a440b74b-85d4-4cb8-9cda-3aff05991549" />








<img width="1460" height="592" alt="Screenshot 2026-07-14 at 02 29 35" src="https://github.com/user-attachments/assets/6200921f-65a1-42db-a7a3-78f4a55e160f" />








<img width="1437" height="589" alt="Screenshot 2026-07-14 at 02 32 26" src="https://github.com/user-attachments/assets/1105ca7e-45c4-4080-a90a-e92b7c49f176" />








<img width="1460" height="588" alt="Screenshot 2026-07-14 at 02 33 45" src="https://github.com/user-attachments/assets/ff78c9b6-6169-4985-b99f-c71826b73417" />








<img width="1163" height="570" alt="Screenshot 2026-07-14 at 02 46 26" src="https://github.com/user-attachments/assets/4a932737-fdec-4db9-a4ef-0cb0c9842bbe" />







<img width="710" height="785" alt="Screenshot 2026-07-14 at 02 46 36" src="https://github.com/user-attachments/assets/30ba9034-b512-4ba4-a67a-ea9076bc0a5e" />
