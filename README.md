# Kubernetes Ansible Playbooks (`k83`)

This folder contains Ansible playbooks and inventory for deploying and managing a Kubernetes cluster using `kubeadm` on Ubuntu 24 hosts. The setup is designed for a single control-plane node and multiple worker nodes, with idempotent operations and safe resets.

## Folder Structure

- [`inventory.ini`](k83/inventory.ini): Ansible inventory file defining the control-plane and worker nodes.
- [`k8s-setup.yaml`](k83/k8s-setup.yaml): Main playbook for preparing all nodes (control-plane and workers) for Kubernetes installation.
- [`k8s-cluster-init.yaml`](k83/k8s-cluster-init.yaml): Playbook for initializing the Kubernetes control-plane and joining worker nodes.
- [`k8s-cluster-reset.yaml`](k83/k8s-cluster-reset.yaml): Playbook for safely resetting the cluster on all nodes.

---

## Inventory

Edit [`inventory.ini`](k83/inventory.ini) to define your cluster nodes:

```ini
[control_plane]
k8smaster ansible_host=192.168.1.200 ansible_user=zoko

[workers]
k8sworker1 ansible_host=192.168.1.201 ansible_user=zoko
k8sworker2 ansible_host=192.168.1.202 ansible_user=zoko

[k8s_cluster:children]
control_plane
workers
```

## Playbooks

### 1. [`k8s-setup.yaml`](k83/k8s-setup.yaml)

**Purpose:**  
Prepares all nodes for Kubernetes by:
- Disabling swap
- Loading required kernel modules
- Setting sysctl parameters
- Installing containerd and Kubernetes packages (`kubelet`, `kubeadm`, `kubectl`)
- Configuring containerd for systemd cgroups

**Run:**
```sh
ansible-playbook -i inventory.ini k8s-setup.yaml
```

---

### 2. [`k8s-cluster-init.yaml`](k83/k8s-cluster-init.yaml)

**Purpose:**  
- Initializes the Kubernetes control-plane (`kubeadm init`)
- Installs Calico CNI for networking
- Generates and distributes the join command to worker nodes
- Joins workers to the cluster

**Run:**
```sh
ansible-playbook -i inventory.ini k8s-cluster-init.yaml
```

---

### 3. [`k8s-cluster-reset.yaml`](k83/k8s-cluster-reset.yaml)

**Purpose:**  
Safely resets all cluster nodes by:
- Stopping kubelet and containerd
- Running `kubeadm reset`
- Removing CNI and Kubernetes state directories
- Flushing iptables

**Run:**
```sh
ansible-playbook -i inventory.ini k8s-cluster-reset.yaml
```

---

## Requirements

- Ansible 2.10+
- Ubuntu 24.x on all nodes
- Passwordless SSH access for the `ansible_user` to all nodes
- Sudo privileges for the `ansible_user`

## Notes

- The playbooks are idempotent and safe to re-run.
- The default pod network CIDR is `10.244.0.0/16` (compatible with Calico).
- Adjust inventory and variables as needed for your environment.

---

## Troubleshooting

- Ensure all nodes have internet access for package installation.
- If you encounter issues with CNI or networking, verify that the pod network CIDR matches your CNI configuration.
- For a clean slate, use the reset playbook before re-initializing the cluster.

---

## References

- [Kubernetes kubeadm documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)
- [Calico CNI](https://docs.tigera.io/calico/latest/getting-started/kubernetes/)

---

*Maintained by Zoko. For questions or improvements, open an issue or PR!*