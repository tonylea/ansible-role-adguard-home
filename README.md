# Ansible Role: AdGuard Home

An Ansible role that installs and configures [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) as a DNS server with ad blocking.

## Requirements

- Ansible >= 2.16
- Target OS: Debian (Trixie)
- Collection dependency: `community.general` (for UFW firewall management)

Install the required collection:

```bash
ansible-galaxy collection install community.general
```

## Role variables

| Variable                        | Default                                          | Description                                                    |
| ------------------------------- | ------------------------------------------------ | -------------------------------------------------------------- |
| `adguard_home_version`          | `v0.107.54`                                      | AdGuard Home version to install                                |
| `adguard_home_install_dir`      | `/opt/AdGuardHome`                               | Directory for the AdGuard Home binary                          |
| `adguard_home_work_dir`         | `/var/lib/AdGuardHome`                           | Working directory for AdGuard Home data                        |
| `adguard_home_config_dir`       | `/etc/AdGuardHome`                               | Directory for AdGuard Home configuration                       |
| `adguard_home_user`             | `adguardhome`                                    | System user to run AdGuard Home                                |
| `adguard_home_group`            | `adguardhome`                                    | System group for AdGuard Home                                  |
| `adguard_home_web_port`         | `3000`                                           | Port for the AdGuard Home web UI                               |
| `adguard_home_dns_port`         | `53`                                             | Port for DNS resolution                                        |
| `adguard_home_firewall_enabled` | `true`                                           | Whether to configure UFW firewall rules                        |
| `adguard_home_arch_map`         | `{x86_64: amd64, aarch64: arm64, armv7l: armv7}` | Maps system architectures to AdGuard Home binary architectures |

## What this role does

1. Creates a dedicated system user and group
2. Downloads and installs the AdGuard Home binary from GitHub releases (with checksum verification)
3. Skips the download if the installed version already matches `adguard_home_version`
4. Disables the `systemd-resolved` stub listener to free port 53 (if applicable)
5. Deploys a systemd service unit with restricted capabilities (`NoNewPrivileges`, `ProtectSystem`, etc.)
6. Opens DNS and web UI ports through UFW (when `adguard_home_firewall_enabled` is `true`)
7. Enables and starts the AdGuard Home service

## Installation

### Ansible Galaxy

```bash
ansible-galaxy role install git+https://github.com/tonylea/ansible-role-adguard-home.git
```

### Manual

Clone the repository into your roles directory:

```bash
git clone https://github.com/tonylea/ansible-role-adguard-home.git roles/adguard_home
```

## Usage

### Basic playbook

```yaml
- hosts: dns_servers
  become: true
  roles:
    - role: adguard_home
```

### Custom version and ports

```yaml
- hosts: dns_servers
  become: true
  roles:
    - role: adguard_home
      adguard_home_version: "v0.107.54"
      adguard_home_web_port: "8080"
      adguard_home_dns_port: "5353"
```

### Without firewall management

```yaml
- hosts: dns_servers
  become: true
  roles:
    - role: adguard_home
      adguard_home_firewall_enabled: false
```

## Tags

Use tags to run specific parts of the role:

| Tag                     | Description                                              |
| ----------------------- | -------------------------------------------------------- |
| `adguard-home`          | All tasks                                                |
| `adguard-home-install`  | User, group, directory creation, and binary installation |
| `adguard-home-config`   | systemd-resolved and service unit configuration          |
| `adguard-home-firewall` | UFW firewall rules                                       |
| `adguard-home-service`  | Service enable and start                                 |

```bash
# Only run firewall tasks
ansible-playbook playbook.yml --tags adguard-home-firewall
```

## Post-install

After the role completes, open the AdGuard Home web UI to finish initial setup:

```
http://<server-ip>:3000
```

## License

MIT