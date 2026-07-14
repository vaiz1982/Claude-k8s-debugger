# claude-k8s-debugger image

Minimal image: Node 22 + kubectl + Claude Code CLI, running as a non-root user.

```bash
docker build -t <your-registry>/claude-k8s-debugger:0.1.0 .
docker push <your-registry>/claude-k8s-debugger:0.1.0
```

Update `chart/values.yaml` with the image reference before deploying.

Two ways to use the running pod:

- **Interactive**: `kubectl exec -it deploy/claude-k8s-debugger -- claude`
  (normal Claude Code session, asks before running risky commands)
- **Headless / one-shot**: override the container command to
  `claude --print --output-format json "your task"` for CI-style invocations.
  Only add `--dangerously-skip-permissions` if you are certain the pod's RBAC
  (see chart/templates/clusterrole.yaml) already limits blast radius — that
  flag removes Claude's own per-command confirmation, so RBAC is your real
  safety net, not the flag.
