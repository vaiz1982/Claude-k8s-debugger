FROM node:22-slim

# kubectl + basic debugging tools
ARG KUBECTL_VERSION=v1.30.3
RUN apt-get update && apt-get install -y --no-install-recommends \
        curl ca-certificates bash git jq \
    && curl -fsSL -o /usr/local/bin/kubectl \
        "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
    && chmod +x /usr/local/bin/kubectl \
    && rm -rf /var/lib/apt/lists/*

# Claude Code CLI
RUN npm install -g @anthropic-ai/claude-code

# Run as non-root
RUN useradd -m -u 10001 claude
USER claude
WORKDIR /workspace

# ANTHROPIC_API_KEY is injected via k8s Secret at runtime (see chart/templates/secret.yaml)
# No key is baked into the image.

ENTRYPOINT ["claude"]
