# Infrastructure Management with Ansible

This repository contains Ansible configurations for managing a heterogeneous server infrastructure, including LDAP authentication setup and standardized sudo configurations across multiple server categories.

I also wrote a guide on setting up Authentik as an LDAP server and for integrating it into Linux [here](https://charlotte.whittleman.net/posts/linux-sso-with-authentik-and-ldap/).

> **🔒 Security Note**: This is a sanitized public version with example configurations. You'll need to customize the example files with your actual infrastructure details before use.

## Overview

This Ansible setup manages the following types of servers:
- **Proxmox Servers**: Virtualization hosts
- **Network Services**: DNS, VPN, reverse proxy, and networking infrastructure
- **Game Servers**: Gaming and entertainment services
- **Development Servers**: Development and testing environments
- **Media Servers**: Media streaming and storage
- **Orchestration Servers**: Container orchestration and management
- **Web Servers**: Web hosting infrastructure
- **Security Servers**: Security-focused services
- **Home Automation**: Home assistant and IoT management

## Repository Structure

```
├── ansible.cfg.example      # Ansible configuration template
├── inventory.example        # Server inventory template with host groups
├── playbook-ldap-setup.yml   # LDAP client configuration playbook
├── playbook-site-sudoers.yml # Sudo configuration deployment playbook
├── templates/
│   └── site-sudoers.j2.example # Jinja2 template for sudoers configuration
└── files/                   # Configuration files for LDAP setup
    ├── common-account
    ├── common-auth
    ├── common-session
    ├── nslcd.conf
    └── nsswitch.conf
```

## Initial Setup

### 1. Create Your Configuration Files

Before using this repository, you need to create your actual configuration files from the examples:

```bash
# Copy example files to actual config files
cp ansible.cfg.example ansible.cfg
cp inventory.example inventory
cp templates/site-sudoers.j2.example templates/site-sudoers.j2
```

### 2. Customize Your Configuration

#### **Inventory Configuration (`inventory`)**
Replace the example values with your actual infrastructure:

```ini
# Replace example IPs (10.0.1.x) with your actual server IPs
pve-host1  ansible_host=YOUR_PROXMOX_IP_1
pve-host2  ansible_host=YOUR_PROXMOX_IP_2

# Replace example server names with your actual hostnames
dns-server         ansible_host=YOUR_DNS_SERVER_IP
vpn-server         ansible_host=YOUR_VPN_SERVER_IP
# ... etc
```

#### **Sudo Configuration (`templates/site-sudoers.j2`)**
Update the user and host aliases:

```bash
# Replace example usernames with your actual admin users
User_Alias   ITADMINS   = your_admin_user1, your_admin_user2
User_Alias   DEVELS     = your_dev_user1, your_dev_user2

# Replace example hostnames with your actual server hostnames
Host_Alias   PROXMOXSERVERS = your-pve-host1, your-pve-host2
Host_Alias   NETSERVERS     = your-dns-server, your-vpn-server, ...
```

#### **Ansible Configuration (`ansible.cfg`)**
Update the SSH key path if different:

```ini
private_key_file = ~/.ssh/your_actual_ssh_key
```

### 3. LDAP Configuration Files

If you're using the LDAP setup, you'll also need to create and customize the following files in the `files/` directory:
- `common-account` - PAM account configuration
- `common-auth` - PAM authentication configuration  
- `common-session` - PAM session configuration
- `nslcd.conf` - NSLCD daemon configuration
- `nsswitch.conf` - Name service switch configuration

## Server Inventory

### Server Groups

The example inventory includes these logical groups:

| Group | Purpose | Example Hosts |
|-------|---------|---------------|
| `proxmoxservers` | Virtualization infrastructure | pve-host1, pve-host2 |
| `netservers` | Network services (DNS, VPN, etc.) | dns-server, vpn-server, proxy-server |
| `gameservers` | Gaming infrastructure | game-server1 |
| `devservers` | Development environments | dev-workstation, dev-server1 |
| `mediaservers` | Media streaming/storage | media-server |
| `orchservers` | Container orchestration | orchestrator1 |

### Network Configuration

- **Example Network**: 10.0.1.x and 10.0.2.x ranges (customize for your network)
- **SSH Key**: `~/.ssh/id_ed25519` (customize as needed)
- **Default User**: root with sudo escalation

## Playbooks

### 1. LDAP Authentication Setup (`playbook-ldap-setup.yml`)

Configures Debian/Ubuntu systems to use LDAP authentication.

**Features:**
- Installs required LDAP client packages (`libpam-ldapd`, `libnss-ldapd`, `libpam-modules`)
- Backs up existing configuration files with timestamps
- Deploys PAM and NSS configurations
- Configures and starts the `nslcd` service

**Usage:**
```bash
ansible-playbook playbook-ldap-setup.yml
```

**Target specific groups:**
```bash
ansible-playbook playbook-ldap-setup.yml --limit netservers
```

### 2. Sudo Configuration (`playbook-site-sudoers.yml`)

Deploys standardized sudo configurations across all servers.

**Features:**
- Ensures sudo package is installed
- Deploys site-wide sudoers configuration from template
- Validates sudoers syntax automatically
- Sets secure permissions (0440)

**Usage:**
```bash
ansible-playbook playbook-site-sudoers.yml
```

## Sudo Configuration Details

The `site-sudoers.j2` template provides:

### Security Features
- No TTY requirement for automation
- Secure PATH enforcement
- 5-minute credential timeout
- Comprehensive logging to `/var/log/sudo.log`
- Input/output logging for audit trails

### User Groups
- **ITADMINS**: Full sudo access with password (customize with your admin users)
- **DEVELS**: Limited access for development tasks (customize with your dev users)

### Host Aliases
Servers are organized into logical groups matching the inventory:
- `PROXMOXSERVERS`, `NETSERVERS`, `GAMESERVERS`, etc.

### Command Aliases
- **PKG_MGMT**: Package management commands (apt, yum, dnf, etc.)
- **SERVICE**: Service management (systemctl, service)
- **NET_TOOLS**: Network troubleshooting tools
- **LOGS**: Log viewing utilities
- **DOCKER**: Docker management
- **SENSITIVE**: Password-protected commands requiring re-authentication

## Prerequisites

### Local Setup
1. Install Ansible on your control machine
2. Generate SSH key: `ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519`
3. Copy SSH key to all target hosts: `ssh-copy-id -i ~/.ssh/id_ed25519 root@<host>`

### Target Host Requirements
- Debian/Ubuntu-based systems
- SSH access configured
- Root access or sudo privileges
- Python installed (for Ansible modules)

## Getting Started

1. **Clone and enter the repository:**
   ```bash
   git clone <repository-url>
   cd inframanagement
   ```

2. **Set up your configuration files:**
   ```bash
   cp ansible.cfg.example ansible.cfg
   cp inventory.example inventory
   cp templates/site-sudoers.j2.example templates/site-sudoers.j2
   ```

3. **Customize the files with your infrastructure details** (see Initial Setup section above)

4. **Verify connectivity:**
   ```bash
   ansible all -m ping
   ```

5. **Deploy sudo configuration:**
   ```bash
   ansible-playbook playbook-site-sudoers.yml
   ```

6. **Configure LDAP authentication (if needed):**
   ```bash
   ansible-playbook playbook-ldap-setup.yml
   ```

## Advanced Usage

### Running on Specific Groups
```bash
# Only network servers
ansible-playbook playbook-ldap-setup.yml --limit netservers

# Only development servers
ansible-playbook playbook-site-sudoers.yml --limit devservers
```

### Dry Run Mode
```bash
ansible-playbook playbook-ldap-setup.yml --check --diff
```

### Verbose Output
```bash
ansible-playbook playbook-site-sudoers.yml -vvv
```

## Security Notes

- All sudo configurations require password authentication
- Sensitive commands (user management, SSH) always require re-authentication
- Configuration backups are created before modifications
- Sudoers syntax is validated automatically before deployment
- All changes are logged for audit purposes
- **Remember**: Customize the example files with your actual infrastructure before use

## Troubleshooting

### SSH Connection Issues
- Verify SSH key permissions: `chmod 600 ~/.ssh/id_ed25519`
- Test manual SSH connection: `ssh -i ~/.ssh/id_ed25519 root@<host>`

### LDAP Issues
- Check nslcd service status: `systemctl status nslcd`
- Review configuration backups in `/etc/*.bak.*`
- Test LDAP connectivity: `getent passwd`

### Sudo Issues
- Validate sudoers manually: `visudo -c`
- Check logs: `tail -f /var/log/sudo.log`
- Review file permissions: `ls -la /etc/sudoers.d/site_sudoers`

### Configuration Issues
- Ensure you've copied and customized all `.example` files
- Verify your inventory file has correct IP addresses and hostnames
- Check that your SSH key path in `ansible.cfg` is correct

## Contributing

When adding new servers:
1. Add entries to the `inventory` file
2. Update host aliases in `templates/site-sudoers.j2` if needed
3. Test connectivity with `ansible <hostname> -m ping`

When contributing to this repository:
- Always use example data in templates
- Never commit real IP addresses, hostnames, or usernames
- Update documentation when adding new features

## License

This infrastructure configuration is designed for educational and operational use. Modify according to your organization's security policies and requirements. 
