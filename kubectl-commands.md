# kubectl Command Reference

## Cluster & Node Management

| Command | Description |
|---------|-------------|
| `kubectl cluster-info` | Display cluster info |
| `kubectl get nodes` | List all nodes |
| `kubectl get nodes -o wide` | List nodes with IPs and roles |
| `kubectl describe node <name>` | Detailed node info |
| `kubectl top nodes` | Node resource usage (requires metrics-server) |
| `kubectl cordon <node>` | Mark node as unschedulable |
| `kubectl uncordon <node>` | Mark node as schedulable |
| `kubectl drain <node> --ignore-daemonsets` | Safely evict pods from a node |

## Pods

| Command | Description |
|---------|-------------|
| `kubectl get pods` | List pods in current namespace |
| `kubectl get pods -A` | List pods in all namespaces |
| `kubectl get pods -o wide` | List pods with IPs and nodes |
| `kubectl describe pod <name>` | Detailed pod info |
| `kubectl logs <pod>` | View pod logs |
| `kubectl logs -f <pod>` | Follow pod logs |
| `kubectl logs -l app=<label>` | Logs from all pods matching label |
| `kubectl exec -it <pod> -- sh` | Shell into a running pod |
| `kubectl port-forward pod/<pod> 8080:80` | Forward local port to pod |
| `kubectl delete pod <pod>` | Delete a pod |

## Deployments

| Command | Description |
|---------|-------------|
| `kubectl create deployment <name> --image=<image>` | Create a deployment |
| `kubectl get deployments` | List deployments |
| `kubectl describe deployment <name>` | Describe a deployment |
| `kubectl scale deployment <name> --replicas=3` | Scale a deployment |
| `kubectl set image deployment/<name> <container>=<new-image>` | Update image |
| `kubectl rollout status deployment/<name>` | Check rollout status |
| `kubectl rollout undo deployment/<name>` | Rollback to previous version |
| `kubectl rollout history deployment/<name>` | View rollout history |
| `kubectl delete deployment <name>` | Delete a deployment |

## Services

| Command | Description |
|---------|-------------|
| `kubectl get svc` | List services |
| `kubectl get svc -o wide` | List services with cluster IPs |
| `kubectl describe svc <name>` | Describe a service |
| `kubectl expose deployment <name> --port=80 --target-port=8080 --type=ClusterIP` | Create a ClusterIP service |
| `kubectl expose deployment <name> --port=80 --type=NodePort` | Create a NodePort service |
| `kubectl expose deployment <name> --port=80 --type=LoadBalancer` | Create a LoadBalancer service (uses MetalLB) |
| `kubectl delete svc <name>` | Delete a service |

## Ingress / Networking

| Command | Description |
|---------|-------------|
| `kubectl get ingress` | List ingress rules |
| `kubectl get svc -A` | List all services across namespaces |
| `kubectl get endpoints` | List service endpoints |

## ConfigMaps & Secrets

| Command | Description |
|---------|-------------|
| `kubectl create configmap <name> --from-file=<file>` | Create ConfigMap from file |
| `kubectl create configmap <name> --from-literal=<key>=<value>` | Create ConfigMap from literal |
| `kubectl get configmaps` | List ConfigMaps |
| `kubectl describe configmap <name>` | Describe a ConfigMap |
| `kubectl create secret generic <name> --from-file=<file>` | Create secret from file |
| `kubectl create secret generic <name> --from-literal=<key>=<value>` | Create secret from literal |
| `kubectl get secrets` | List secrets |
| `kubectl describe secret <name>` | Describe a secret |

## Namespaces

| Command | Description |
|---------|-------------|
| `kubectl get namespaces` | List namespaces |
| `kubectl create namespace <name>` | Create a namespace |
| `kubectl delete namespace <name>` | Delete a namespace |

## Resource Monitoring

| Command | Description |
|---------|-------------|
| `kubectl get events -A --sort-by='.lastTimestamp'` | Cluster events sorted by time |
| `kubectl get events -A --watch` | Watch events in real-time |
| `kubectl top pods` | Pod CPU/memory usage |
| `kubectl top nodes` | Node CPU/memory usage |

## Applying & Deleting Resources

| Command | Description |
|---------|-------------|
| `kubectl apply -f <file.yaml>` | Apply resource from file |
| `kubectl apply -f <directory/>` | Apply all resources in directory |
| `kubectl delete -f <file.yaml>` | Delete resource from file |
| `kubectl delete pod,svc,deployment --all` | Delete all resources in namespace |

## Useful Shortcuts

```bash
# Alias
alias k=kubectl

# Watch pods continuously
kubectl get pods -w

# Get all resources in namespace
kubectl get all

# Pretty-print YAML output
kubectl get pod <name> -o yaml

# Quick pod shell
kubectl exec -it deploy/<name> -- sh

# Delete all failed pods
kubectl delete pods --field-selector=status.phase=Failed

# Show resource usage with labels
kubectl top pods --containers
```

## K3s-Specific

Since kubectl is symlinked to k3s on the server nodes, you can also run:

```bash
k3s kubectl get nodes        # via k3s binary (on server nodes)
kubectl get nodes            # via standalone kubectl (from yoshi)
```
