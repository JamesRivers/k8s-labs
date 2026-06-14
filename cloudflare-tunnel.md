# Cloudflare Tunnel Setup

This document explains how Cloudflare Tunnel was set up to expose cluster services without opening ports on the home router.

## Overview

Cloudflare Tunnel creates an outbound connection from the cluster to Cloudflare's edge network. Traffic hits Cloudflare first, then routes through the tunnel to your services — no public IP or port forwarding needed.

## Architecture

```
User --> Cloudflare Edge --> Tunnel (QUIC) --> cloudflared Pod --> Service (ClusterIP)
```

## Components

| Component | Location | Purpose |
|-----------|----------|---------|
| `cloudflared` CLI | moe (server) | Create and manage tunnels |
| `cloudflared` Pod | K3s cluster | Runs the tunnel connection |
| Tunnel credentials | K8s Secret | Authenticates the tunnel to Cloudflare |
| ConfigMap | K8s ConfigMap | Defines ingress routing rules |

## Setup Steps

### 1. Install cloudflared

On the K3s server node (moe):

```bash
curl -sfL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared
```

### 2. Authenticate

```bash
cloudflared tunnel login
```

Opens a URL to authenticate with Cloudflare. The cert is saved to `~/.cloudflared/cert.pem`.

### 3. Create a tunnel

```bash
cloudflared tunnel create k3s-lab
```

Creates a tunnel and saves credentials JSON to `~/.cloudflared/<uuid>.json`.

### 4. Route DNS

```bash
cloudflared tunnel route dns k3s-lab wyc-lab.com
```

Creates a CNAME in Cloudflare DNS pointing the domain to the tunnel.

### 5. Create tunnel config

```yaml
# config.yaml
tunnel: k3s-lab
credentials-file: /etc/cloudflared/credentials.json

ingress:
  - hostname: wyc-lab.com
    service: http://nginx-test.default.svc.cluster.local:80
  - hostname: www.wyc-lab.com
    service: http://nginx-test.default.svc.cluster.local:80
  - service: http_status:404
```

### 6. Deploy in Kubernetes

Create the credentials secret and config:

```bash
kubectl create secret generic tunnel-credentials \
  --from-file=credentials.json=credentials.json

kubectl create configmap tunnel-config --from-file=config.yaml
```

Deploy cloudflared as a pod:

```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloudflared
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cloudflared
  template:
    metadata:
      labels:
        app: cloudflared
    spec:
      containers:
      - name: cloudflared
        image: cloudflare/cloudflared:latest
        args:
        - tunnel
        - --config
        - /etc/cloudflared/config/config.yaml
        - run
        volumeMounts:
        - name: config
          mountPath: /etc/cloudflared/config
        - name: credentials
          mountPath: /etc/cloudflared
      volumes:
      - name: config
        configMap:
          name: tunnel-config
      - name: credentials
        secret:
          secretName: tunnel-credentials
EOF
```

### 7. Verify

Check the tunnel pod logs:

```bash
kubectl logs -l app=cloudflared
```

Test the endpoint:

```bash
curl http://wyc-lab.com
```

## Adding More Services

Edit the config.yaml to add more hostname/service entries:

```yaml
ingress:
  - hostname: app.wyc-lab.com
    service: http://my-app.default.svc.cluster.local:8080
  - hostname: wyc-lab.com
    service: http://nginx-test.default.svc.cluster.local:80
  - service: http_status:404
```

Update the ConfigMap:

```bash
kubectl create configmap tunnel-config --from-file=config.yaml --dry-run=client -o yaml | kubectl apply -f -
```

Restart the cloudflared pod to pick up changes.

## Troubleshooting

- **DNS not resolving**: Ensure the domain nameservers point to Cloudflare and the CNAME record was created.
- **Tunnel won't start**: Check credentials secret exists and matches the config's tunnel ID.
- **502 Bad Gateway**: The service hostname is not reachable from within the cluster — verify the service name and port.
