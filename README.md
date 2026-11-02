# Linux Server Bootstrap - Ansible Port

This is an Ansible port of [Linux-Server-Bootstrap](../Linux-Server-Bootstrap), converting the bash-based hardening tool to Ansible playbooks while preserving the same architecture and security posture.

## Features

- **Multi-distro support**: Debian/Ubuntu, RHEL/Rocky/AlmaLinux/Fedora/Amazon Linux, openSUSE Leap/SLES
- **13 ordered stages**: Same stage order as original (base, users, ssh, firewall, fail2ban, sysctl, limits, perms, audit, mac, services, banner, auth)
- **Family-specific drivers**: Different implementations for apt/ufw, yum/dnf/firewalld, zypper/firewalld
- **Backups & verification**: Creates backups before changes, state tracking
- **Idempotent**: Safe to run multiple times
- **Check mode support**: Test with `--check --diff` before applying

## Quick Start

1. **Configure inventory**:
```bash
cp inventory/hosts.example inventory/hosts
# Edit inventory/hosts with your servers
```

2. **Set variables** in `group_vars/all.yml` or pass via `-e`:
```bash
ansible-playbook site.yml -i inventory/hosts -e "bootstrap_ssh_users=['deploy'] bootstrap_ssh_port=22"
```

3. **Dry run first**:
```bash
ansible-playbook site.yml -i inventory/hosts --check --diff
```

4. **Apply**:
```bash
ansible-playbook site.yml -i inventory/hosts
```

5. **Verify**:
```bash
ansible-playbook verify.yml -i inventory/hosts
```

## Stages

| Stage | Description | Tags |
|-------|-------------|------|
| base | Package updates, hostname, timezone, locale, time sync | base |
| users | Create accounts, install keys, sudo access | users |
| ssh | SSH hardening (disable password/root login, key-only) | ssh |
| firewall | Configure firewall (ufw/firewalld) | firewall |
| fail2ban | Install and configure fail2ban | fail2ban |
| sysctl | Kernel hardening parameters | sysctl |
| limits | Login/core limits | limits |
| perms | File permissions hardening | perms |
| audit | auditd rules | audit |
| mac | SELinux/AppArmor | mac |
| services | Disable legacy services | services |
| banner | Login banners | banner |
| auth | Authentication policy (PAM, faillock, pwquality) | auth |

## Usage Examples

Run specific stages:
```bash
ansible-playbook site.yml -i inventory/hosts --tags ssh,firewall
```

Skip stages:
```bash
ansible-playbook site.yml -i inventory/hosts -e "bootstrap_skip=['fail2ban']"
```

Create new user:
```bash
ansible-playbook site.yml -i inventory/hosts -e "bootstrap_new_users=['deploy'] bootstrap_user_keys=['ssh-rsa AAAA...'] bootstrap_sudo=true"
```

## Safety Notes

- Always run with `--check --diff` first
- Ensure SSH key is in place for the allow user before disabling password auth
- Keep a separate SSH session open during first run
- The playbook includes guards against locking yourself out
- Backups are stored in `/var/backups/linux-server-bootstrap/`

## Differences from Bash Version

- Uses Ansible idempotency (safe to re-run)
- Check mode support built-in
- Better for managing multiple servers
- State tracked via facts and files
- Rollback via `rollback.yml` playbook
