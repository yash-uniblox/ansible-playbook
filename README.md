# Ansible — Single Server K3s & Ollama Setup

This project automates the installation of **K3s, Ollama, and Traefik** on a single Linux server using Ansible.

## Architecture

```text
Ansible Master
      |
      | SSH
      v
Target Server
 ├── K3s
 ├── Ollama
 └── Traefik
```

## Prerequisites

- Ansible Master: Ubuntu/Debian Linux
- Target Server: Ubuntu/Debian Linux
- SSH access to the target server
- `sudo` privileges on the target server

---

## 1. Install Ansible

On the **Ansible Master**:

```bash
sudo apt update
sudo apt install ansible -y
```

Verify the installation:

```bash
ansible --version
```

---

## 2. Create the Project

```bash
mkdir ansible-k3s
cd ansible-k3s
```

Project structure:

```text
ansible-k3s/
├── inventory.ini
└── playbook.yaml
```

---

## 3. Generate SSH Key

On the Ansible Master:

```bash
ssh-keygen -t ed25519
```

This creates:

```text
~/.ssh/id_ed25519       # Private key
~/.ssh/id_ed25519.pub   # Public key
```

> **Important:** Never share or commit the private key.

---

## 4. Configure Inventory

Create the inventory file:

```bash
nano inventory.ini
```

Add the target server:

```ini
[k3s_server]
server1 ansible_host=192.168.1.20 ansible_user=ubuntu
```

Replace the IP address and username with your target server details.

---

## 5. Configure SSH Access

Copy the public key to the target server:

```bash
ssh-copy-id ubuntu@192.168.1.20
```

Test the SSH connection:

```bash
ssh ubuntu@192.168.1.20
```

Exit the server:

```bash
exit
```

---

## 6. Test Ansible Connectivity

From the project directory:

```bash
ansible all -i inventory.ini -m ping
```

Expected output:

```text
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## 7. Create the Playbook

Create the Ansible playbook:

```bash
nano playbook.yaml
```

The `playbook.yaml` installs and configures:

- Required system packages
- Ollama
- K3s
- Ollama systemd service
- K3s systemd service
- Traefik verification

---

## 8. Run the Playbook

Run:

```bash
ansible-playbook -i inventory.ini playbook.yaml
```

If sudo requires a password:

```bash
ansible-playbook -i inventory.ini playbook.yaml --ask-become-pass
```

---

## 9. Verify the Installation

SSH into the target server:

```bash
ssh ubuntu@192.168.1.20
```

### Verify K3s

```bash
sudo k3s kubectl get nodes
```

The node should have:

```text
STATUS
Ready
```

### Verify Ollama

```bash
ollama --version
```

Check the service:

```bash
sudo systemctl status ollama
```

### Verify Traefik

```bash
sudo k3s kubectl get deployment traefik -n kube-system
```

### Verify Kubernetes Pods

```bash
sudo k3s kubectl get pods -A
```

---

## Deployment Flow

```text
Install Ansible
      ↓
Generate SSH Key
      ↓
Configure inventory.ini
      ↓
Exchange SSH Keys
      ↓
Test Ansible Connection
      ↓
Run playbook.yaml
      ↓
K3s + Ollama + Traefik Installed
      ↓
Verify Installation
```

## Security

- Never commit private SSH keys or secrets.
- Do not expose the Kubernetes API (`6443`) publicly unless required.
- Use Ansible Vault for sensitive credentials or tokens.