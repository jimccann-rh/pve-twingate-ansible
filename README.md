# Proxmox Twingate Connector Ansible Playbook

This Ansible playbook automates the deployment of a Twingate network connector by creating a new Linux Container (LXC) on Proxmox Virtual Environment (PVE) and installing the Twingate connector software.

## Overview

The playbook performs the following operations:

1. **Container Creation**: Creates a new LXC container on a Proxmox host with customizable specifications
2. **System Setup**: Updates the container, installs required packages (curl, sudo)
3. **Twingate Installation**: Installs and configures the Twingate network connector

## What is Twingate?

Twingate is a zero-trust network access solution that provides secure remote access to your network resources. This playbook creates a connector that acts as a bridge between your Twingate network and your local infrastructure.

## Prerequisites

### System Requirements
- Ansible installed on your control machine
- Python `proxmoxer` library (automatically installed by the playbook)
- SSH access from your control machine to the Proxmox host
- A Proxmox VE environment with:
  - Available VM ID for the new container
  - Container template (Debian 12 by default)
  - Sufficient resources (CPU, RAM, storage)

### Twingate Requirements
- A Twingate account and network
- Twingate connector tokens (Access Token and Refresh Token)

## Configuration

### Proxmox Settings

Edit the following variables in `cpcvm.yml` under the `vars` section:

| Variable | Description | Example |
|----------|-------------|---------|
| `proxmox_host` | IP address or hostname of your Proxmox server | `"192.168.1.10"` |
| `proxmox_user` | Proxmox username with API access | `"root@pam"` |
| `proxmox_password` | Proxmox user password | `"your_password"` |
| `proxmox_node` | Proxmox node name | `"pve"` |

### Container Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `container_vmid` | Unique VM ID for the container | `8888` |
| `container_hostname` | Hostname for the container | `"twingate"` |
| `container_template` | LXC template to use | `"local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst"` |
| `container_memory_mb` | RAM allocation in MB | `512` |
| `container_swap_mb` | Swap space in MB | `512` |
| `container_disk_gb` | Root disk size in GB | `1` |
| `container_storage` | Proxmox storage backend | `"local-lvm"` |
| `container_cores` | Number of CPU cores | `2` |

### Network Settings

| Variable | Description | Example |
|----------|-------------|---------|
| `container_network_ip` | IP address with CIDR notation | `"192.168.1.101/24"` |
| `container_network_gateway` | Network gateway | `"192.168.1.1"` |
| `container_network_bridge` | Proxmox network bridge | `"vmbr0"` |
| `container_dns_servers` | DNS server IP | `"192.168.1.1"` |

## Usage

### 1. Install Ansible
```bash
# On Ubuntu/Debian
sudo apt update && sudo apt install ansible

# On CentOS/RHEL
sudo yum install ansible

# Using pip
pip install ansible
```

### 2. Configure Twingate Environment Variable

1. Log into your Twingate Admin Console
2. Navigate to **Settings** → **Connectors**
3. Click **Add Connector**
4. Select **Script-based deployments** → **Linux**
5. Generate tokens and copy the provided command
6. Export it as an environment variable:

```bash
export TWINGATE='curl "https://binaries.twingate.com/connector/setup.sh" | sudo TWINGATE_ACCESS_TOKEN="your_access_token" TWINGATE_REFRESH_TOKEN="your_refresh_token" TWINGATE_NETWORK="your_network_name" TWINGATE_LABEL_DEPLOYED_BY="linux" bash'
```

### 3. Edit Configuration

Edit the variables in `cpcvm.yml` to match your environment:

```bash
nano cpcvm.yml
```

**Important**: Update at least these critical variables:
- `proxmox_host`
- `proxmox_password`
- `container_network_ip`
- `container_network_gateway`
- `container_dns_servers`

### 4. Run the Playbook

```bash
ansible-playbook cpcvm.yml
```

## Security Considerations

⚠️ **Important Security Notes**:

1. **Passwords in Plain Text**: The current configuration stores passwords in plain text. For production use, consider:
   - Using Ansible Vault to encrypt sensitive variables
   - Using SSH key authentication instead of passwords

2. **Certificate Validation**: The playbook disables SSL certificate validation (`validate_certs: no`). In production:
   - Use valid SSL certificates
   - Set `validate_certs: yes`

3. **SSH Key Authentication**: The playbook includes SSH public key setup. Ensure your SSH key exists:
   ```bash
   ls ~/.ssh/id_rsa.pub
   ```

### Using Ansible Vault (Recommended)

Encrypt sensitive variables:
```bash
ansible-vault create secrets.yml
```

Add encrypted variables to your secrets file and reference them in the main playbook.

## Container Features

The created container includes:
- **Unprivileged**: Runs as an unprivileged container for better security
- **Nesting**: Enables container nesting (`nesting=1`) if needed for certain applications
- **Auto-start**: Configured to start automatically when Proxmox boots
- **SSH Access**: SSH server enabled with key-based authentication

## Troubleshooting

### Common Issues

1. **VM ID Already Exists**
   - Change `container_vmid` to an unused ID
   - Check Proxmox UI for available IDs

2. **Network Connectivity Issues**
   - Verify IP address is available and in correct subnet
   - Check gateway and DNS settings
   - Ensure network bridge exists on Proxmox

3. **Template Not Found**
   - Download the required LXC template in Proxmox
   - Update `container_template` variable with correct path

4. **Twingate Installation Fails**
   - Verify `TWINGATE` environment variable is set correctly
   - Check internet connectivity from the container
   - Ensure tokens are valid and not expired

### Debugging

Enable verbose output:
```bash
ansible-playbook -vvv cpcvm.yml
```

Check container logs in Proxmox:
```bash
pct list
pct enter <VMID>
```

## Maintenance

### Updating the Container
The Twingate connector will auto-update, but you can manually update the system:

```bash
pct enter <VMID>
apt update && apt upgrade -y
```

### Monitoring
Monitor the connector status in your Twingate Admin Console under **Settings** → **Connectors**.

## License

This project is licensed under the terms specified in the LICENSE file.

