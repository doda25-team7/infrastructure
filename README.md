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

- Control node: `192.168.68.100`
- Worker node-N: `192.168.68.{100+N}`
- Ingress controller: `192.168.68.90`

## Usage

### Start all VMs
```bash
vagrant up
```

### Finalizing the Cluster Setup

```bash
ansible-playbook -i ansible/inventory.ini ansible/finalization.yaml
```
### If KVM is conflicting temporarily disable it 
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

## Check the network status
log in the ctrl node and do `kubectl get nodes`

## Access the Dashboard

### Step 1: Add dashboard.local to your hosts file

**On macOS/Linux:**
```bash
sudo nano /etc/hosts
```
Add this line at the end of the file:
```
192.168.68.90    dashboard.local
```
Save and exit (Ctrl+X, then Y, then Enter in nano).

**On Windows:**
1. Open Notepad as Administrator
2. Open `C:\Windows\System32\drivers\etc\hosts`
3. Add this line at the end:
```
192.168.68.90    dashboard.local
```
4. Save the file

### Step 2: Verify the cluster is ready

First, make sure your VMs are running and the finalization playbook has been executed:
```bash
vagrant status
```

If the finalization hasn't been run yet, execute it:
```bash
cd infrastructure
ansible-playbook -i ansible/inventory.ini ansible/finalization.yaml
```

### Step 3: Get the authentication token

SSH into the control node:
```bash
vagrant ssh ctrl
```

Once inside the ctrl VM, generate the token:
```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Copy the entire token that is displayed (it will be a long string).

### Step 4: Access the dashboard

1. Open your web browser
2. Navigate to: `http://dashboard.local/`
3. You should see a login page asking for a token
4. Paste the token you copied in Step 3
5. Click "Sign in"

### Troubleshooting

**If `http://dashboard.local/` doesn't work:**

1. First, verify the hosts entry:
   ```bash
   # On macOS/Linux
   cat /etc/hosts | grep dashboard.local
   
   # Should show: 192.168.68.90    dashboard.local
   ```

2. Try accessing by IP directly:
   ```bash
   # Test if the IP is reachable
   ping 192.168.68.90
   ```

3. If ping fails, check if the ingress controller is running:
   ```bash
   vagrant ssh ctrl
   kubectl get pods -n ingress-nginx
   kubectl get svc -n ingress-nginx
   ```

4. If needed, manually add the IP to the control node's interface:
   ```bash
   vagrant ssh ctrl
   sudo ip addr add 192.168.68.90/24 dev eth1
   ```

5. Verify the dashboard ingress is configured:
   ```bash
   vagrant ssh ctrl
   kubectl get ingress -n kubernetes-dashboard
   ```

## NOTES
login the ctrl and use `systemctl restart` to restart failed service of any service failed in k8s