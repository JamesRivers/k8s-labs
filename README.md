# K3s Lab Cluster Topology

```mermaid
graph TB
    subgraph yoshi["yoshi (local machine)"]
        Y_CPU["CPU: Apple Silicon (aarch64)"]
        Y_ROLE["Role: kubectl control plane client"]
    end

    subgraph moe["moe - 192.168.0.122"]
        M_CPU["CPU: Intel Pentium Silver J5005 @ 1.50GHz (4 cores)"]
        M_RAM["RAM: 7.6 GiB"]
        M_DISK["Disk: 32GB SATA Flash + 1TB USB HDD"]
        M_ROLE["Role: K3s Server (control-plane + etcd)"]
    end

    subgraph larry["larry - 192.168.0.124"]
        L_CPU["CPU: Intel Pentium Silver J5005 @ 1.50GHz (4 cores)"]
        L_RAM["RAM: 7.6 GiB"]
        L_DISK["Disk: 32GB SATA Flash"]
        L_ROLE["Role: K3s Agent"]
    end

    subgraph curly["curly - 192.168.0.125"]
        C_CPU["CPU: Intel Pentium Silver J5005 @ 1.50GHz (4 cores)"]
        C_RAM["RAM: 7.6 GiB"]
        C_DISK["Disk: 32GB SATA Flash"]
        C_ROLE["Role: K3s Agent"]
    end

    subgraph metallb["MetalLB"]
        LB_POOL["IP Pool: 192.168.0.200-220"]
        LB_MODE["Mode: Layer 2"]
    end

    yoshi -- "kubectl (kubeconfig)" --> moe
    moe -- "control plane" --> larry
    moe -- "control plane" --> curly
    metallb -- "LoadBalancer IPs" --> moe
    metallb -- "LoadBalancer IPs" --> larry
    metallb -- "LoadBalancer IPs" --> curly
```

## Hosts

| Hostname | IP | Role | CPU | RAM | Disk |
|----------|----|------|-----|-----|------|
| moe | 192.168.0.122 | K3s server (control-plane + etcd) | Intel Pentium Silver J5005 (4C) | 7.6 GiB | 32GB + 1TB USB |
| larry | 192.168.0.124 | K3s agent | Intel Pentium Silver J5005 (4C) | 7.6 GiB | 32GB |
| curly | 192.168.0.125 | K3s agent | Intel Pentium Silver J5005 (4C) | 7.6 GiB | 32GB |
| yoshi | local | kubectl client | Apple Silicon (aarch64) | - | - |

## Networking

| Component | Details |
|-----------|---------|
| **MetalLB** | Bare-metal load balancer, Layer 2 mode |
| **IP Pool** | `192.168.0.200` – `192.168.0.220` |
| **CNI** | Flannel (K3s default) |
| **Pod CIDR** | `10.42.0.0/16` |
| **Service CIDR** | `10.43.0.0/16` |

## Storage

- **/wyc-kubernetes-labs** on moe — 1TB USB HDD mounted at `/wyc-kubernetes-labs` for config backups and documentation
