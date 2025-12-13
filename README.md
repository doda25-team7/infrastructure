# Vagrant Kubernetes Setup

This directory contains the Vagrant configuration for setting up a Kubernetes cluster with VirtualBox.

## Prerequisites

- [Vagrant](https://www.vagrantup.com/downloads) installed 
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed

```bash
sudo apt update
sudo apt install virtualbox
```

## Test installations
```bash
vagrant --version
vboxmanage --version
```

## Configuration

The setup creates the following VMs:

- **ctrl**: Control node (1 core, 4GB RAM) - Kubernetes controller
- **node-1, node-2, ...**: Worker nodes (2 cores, 6GB RAM each) - Kubernetes workers

### Scaling Workers

To change the number of worker nodes, edit the `NUM_WORKERS` variable in the `Vagrantfile`:

```ruby
NUM_WORKERS = 2  # Change this number to scale up or down
```

## Network Configuration

- Control node: `192.168.56.100`
- Worker node-N: `192.168.56.{100+N}`

## Usage

### Start all VMs
```bash
vagrant up
```
## If KVM is conflicting temporarily disable it 
```bash
sudo modprobe -r kvm_intel kvm
```

### Start specific VM
```bash
vagrant up ctrl
vagrant up node-1
```

### SSH into a VM
```bash
vagrant ssh ctrl
vagrant ssh node-1
```

### Check VM status
```bash
vagrant status
```

### Stop VMs
```bash
vagrant halt
```

### Destroy VMs
```bash
vagrant destroy -f
```

### Reload VMs (after config changes)
```bash
vagrant reload
```

## Resource Adjustments

Resources and VM configs can be adjusted in the `Vagrantfile`:

- For control node: Modify `vb.memory` and `vb.cpus` in the `ctrl` block
- For worker nodes: Modify `vb.memory` and `vb.cpus` in the worker node block

# Kubernetes Setup

## Kubernetes vs Docker Compose

| Docker Compose | Kubernetes |
|----------------|------------|
| Single machine | Multiple machines |
| Manual restart | Auto-restart |
| No load balancing | Built-in |
| Port mapping | Ingress routing |

Docker is good for small projects that run on one machine, but when more complexity, load balancing and failure recovery is needed kubernetes is the way to go.

## What Each Resource Does

### Deployment
Runs and manages your containers (Pods). If a Pod crashes, Deployment automatically restarts it. It's like a supervisor that keeps your the app running.

### Service
Gives the Pods a stable DNS name. Pods have changing IPs, but Service provides a permanent address like `http://model-service:8081`.

### Ingress
Routes external traffic into the cluster. Maps `http://doda.local` → `app-service`.

### ConfigMap
Stores configuration (non-sensitive data) like URLs, ports, feature flags. One ConfigMap can be used by multiple Pods.

### Secrets 
TODO

## How They Connect

```
User → Ingress (doda.local) → Service (app-service) → Deployment / ConfigMap (config)   → Pod (container)                                                                                         
```

## Quick Commands

```bash
# Deploy everything
kubectl apply -f kubernetes-basic.yaml

# Check status
kubectl get deployments,services,ingress,pods

# View logs
kubectl logs -l app=app-service

# Scale up
kubectl scale deployment app-service --replicas=3
```