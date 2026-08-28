# Ansible Linux Collection

A collection of Ansible roles and modules for Linux system administration.

## Description

This collection provides a curated set of roles for managing Linux systems, covering user management, system upgrades, kernel configuration, container runtimes, security hardening, networking, and storage (swap). All roles are vendored as git submodules from [iamenr0s/ansible-role-*](https://github.com/iamenr0s) and operate across the major Linux distributions (AlmaLinux, Rocky Linux, Fedora, Debian, Ubuntu).

## Installation

### Install the Collection

```bash
ansible-galaxy collection install iamenr0s.ansible_linux_collection
```

### Initialize Git Submodules

This collection includes all roles as git submodules. After cloning the collection, initialize the submodules:

```bash
git submodule update --init --recursive
```

## Included Roles (via git submodules)

This collection bundles the following roles under `roles/`. Each is a standalone git submodule and can also be used independently.

### Table of Contents

| Role | Purpose | Source |
|------|---------|--------|
| `ansible-role-upgrade` | Safe system-wide package upgrades with pre-checks and reboot | [repo](https://github.com/iamenr0s/ansible-role-upgrade) |
| `ansible-role-swap` | Toggle swap on/off and tune `vm.swappiness` | [repo](https://github.com/iamenr0s/ansible-role-swap) |
| `ansible-role-cri-o` | Install and configure CRI-O container runtime | [repo](https://github.com/iamenr0s/ansible-role-cri-o) |
| `ansible-role-etc-hosts` | Declaratively manage `/etc/hosts` (localhost + inventory block) | [repo](https://github.com/iamenr0s/ansible-role-etc-hosts) |
| `ansible-role-firewalld` | Install and manage `firewalld`, zones, rules, and SELinux | [repo](https://github.com/iamenr0s/ansible-role-firewalld) |
| `ansible-role-containerd` | Install and configure containerd with systemd cgroup driver | [repo](https://github.com/iamenr0s/ansible-role-containerd) |
| `ansible-role-users` | Manage system users, groups, SSH keys, and sudo | [repo](https://github.com/iamenr0s/ansible-role-users) |
| `ansible-role-ssh-hardening` | Hardened OpenSSH server/client configuration | [repo](https://github.com/iamenr0s/ansible-role-ssh-hardening) |
| `ansible-role-kernel-configuration` | Distro-aware `sysctl` and kernel module management | [repo](https://github.com/iamenr0s/ansible-role-kernel-configuration) |
| `ansible-role-cgroup` | Enforce cgroup v1/v2 via GRUB-persistent kernel parameters | [repo](https://github.com/iamenr0s/ansible-role-cgroup) |

---

#### ansible-role-upgrade

**Source**: [https://github.com/iamenr0s/ansible-role-upgrade](https://github.com/iamenr0s/ansible-role-upgrade)  
**Location**: `roles/ansible-role-upgrade/`

Performs safe, system-wide package upgrades on Debian/Ubuntu, RHEL-family (AlmaLinux/RockyLinux), and Fedora hosts. Includes pre-checks (OS support, uptime, load average), per-package-manager upgrade routines, and post-checks that reboot when required.

**Key Features**:
- Validates OS support against a curated stable list (`upgrade_stable_os`)
- Checks minimum uptime and load average before proceeding
- Supports `apt`, `dnf`, and `yum` flows with cache management
- Holds packages across the upgrade (`upgrade_packages_on_hold`)
- Detects reboot hints and performs a controlled reboot, then waits for the host

**Example Usage**:
```yaml
- hosts: all
  become: true
  roles:
    - role: ansible-role-upgrade
```

---

#### ansible-role-swap

**Source**: [https://github.com/iamenr0s/ansible-role-swap](https://github.com/iamenr0s/ansible-role-swap)  
**Location**: `roles/ansible-role-swap/`

Manages the system swap runtime state across Linux hosts. Toggles swap on/off via `swapon`/`swapoff` and can optionally tune `vm.swappiness`. Skips swap toggling inside containers.

**Key Features**:
- Enables or disables swap cleanly and idempotently
- Optional tuning of `vm.swappiness` via `ansible.posix.sysctl`
- Container-aware: skips swap toggling in container/virtualized environments
- Cross-distro support (AlmaLinux, Debian, Fedora, Rocky, Ubuntu)

**Example Usage**:
```yaml
- hosts: all
  become: true
  roles:
    - role: ansible-role-swap
      vars:
        swap_enabled: true
        swap_swappiness: 10
```

---

#### ansible-role-cri-o

**Source**: [https://github.com/iamenr0s/ansible-role-cri-o](https://github.com/iamenr0s/ansible-role-cri-o)  
**Location**: `roles/ansible-role-cri-o/`

Automates the installation and configuration of [CRI-O](https://cri-o.io/) on RHEL-family, Fedora, Debian, and Ubuntu hosts. Adds the upstream OBS repository, configures drop-ins in `/etc/crio/crio.conf.d/`, and manages the crio systemd service.

**Key Features**:
- Adds the upstream OBS `cri-o` repository (yum/dnf or APT, matching `crio_version`)
- Installs and configures CRI-O including its drop-in directory
- Optionally installs and configures `crun` as the container runtime
- Optionally pins the crio systemd service to a non-default systemd slice
- Manages arbitrary extra CRI-O configuration via `crio_extra_config`

**Example Usage**:
```yaml
- hosts: k8s_nodes
  become: true
  roles:
    - role: ansible-role-cri-o
```

---

#### ansible-role-etc-hosts

**Source**: [https://github.com/iamenr0s/ansible-role-etc-hosts](https://github.com/iamenr0s/ansible-role-etc-hosts)  
**Location**: `roles/ansible-role-etc-hosts/`

Declaratively manages `/etc/hosts` using only OS-agnostic `ansible.builtin` modules. Updates the `127.0.0.1` line to include the FQDN/hostname and maintains an idempotent block of host mappings derived from the inventory.

**Key Features**:
- Configurable localhost line: FQDN, short hostname, `localhost`, extra aliases
- Inventory-derived `/etc/hosts` block, optionally restricted to specific inventory groups
- Static extra entries via `etc_hosts_entries`
- Idempotent, marker-delimited managed block (`# BEGIN/END ANSIBLE MANAGED HOSTS`)
- No per-distro branching — uses only `lineinfile`/`blockinfile`

**Example Usage**:
```yaml
- hosts: all
  roles:
    - role: ansible-role-etc-hosts
```

---

#### ansible-role-firewalld

**Source**: [https://github.com/iamenr0s/ansible-role-firewalld](https://github.com/iamenr0s/ansible-role-firewalld)  
**Location**: `roles/ansible-role-firewalld/`

Manages `firewalld` installation, service state, kernel parameters/modules, and per-zone configuration. Integrates with package-management and kernel-configuration roles.

**Key Features**:
- Ensures required `firewalld` packages are present
- Optionally manages kernel modules and sysctl settings required by firewalld
- Applies services, ports (single/range), rich rules, sources, and interfaces per zone
- Configures masquerade, ICMP blocks, and port forwarding
- Optionally manages SELinux state and booleans
- Reloads after zone creation

**Example Usage**:
```yaml
- hosts: all
  become: true
  roles:
    - role: ansible-role-firewalld
```

> **Note**: This role integrates with `iamenr0s.ansible_role_pkg_management` and `iamenr0s.ansible_role_kernel_configuration` (installable from Galaxy or vendored in this collection under `roles/`).

---

#### ansible-role-containerd

**Source**: [https://github.com/iamenr0s/ansible-role-containerd](https://github.com/iamenr0s/ansible-role-containerd)  
**Location**: `roles/ansible-role-containerd/`

Installs and configures [containerd](https://containerd.io/) from the Docker CE repository on RHEL-family (AlmaLinux/RockyLinux), Fedora, and Debian-family (Debian/Ubuntu) hosts. Writes containerd's default configuration and can switch the cgroup driver to systemd (the recommended setup for Kubernetes).

**Key Features**:
- Adds the Docker CE repository and GPG key (RedHat and Debian families)
- Installs `containerd.io` (and `container-selinux` on RedHat-family)
- Manages service state and boot enablement
- Optionally writes default config to `/etc/containerd/config.toml`
- Optionally sets `SystemdCgroup = true`
- Restarts containerd only when the configuration actually changes

**Example Usage**:
```yaml
- hosts: k8s_nodes
  become: true
  roles:
    - role: ansible-role-containerd
      vars:
        containerd_config_default: true
        containerd_config_cgroup_driver_systemd: true
```

---

#### ansible-role-users

**Source**: [https://github.com/iamenr0s/ansible-role-users](https://github.com/iamenr0s/ansible-role-users)  
**Location**: `roles/ansible-role-users/`

Manages the addition of users and groups to your system. Comprehensive user management including SSH key management, sudo configuration, and password handling.

**Key Features**:
- Home directory management
- SSH key generation and deployment
- Flexible sudo permissions
- Password expiration settings
- Group membership management
- Ansible Vault integration for sensitive data

**Example Usage**:
```yaml
- name: Manage users and groups
  hosts: all
  roles:
    - role: ansible-role-users
      users_groups:
        - name: developers
          gid: 1024
        - name: admins
          system: true
      users:
        - name: john_doe
          comment: "John Doe - Developer"
          uid: 1024
          group: developers
          groups:
            - admins
          manage_ssh_key: true
          sudo_options: "ALL=(ALL) NOPASSWD: ALL"
          authorized_keys:
            - "ssh-rsa AAAAB3NzaC1yc2E..."
        - name: service_user
          system: true
          manage_ssh_key: false
```

**Variables**:
- `default_user_shell`: Default shell for users (default: `/bin/bash`)
- `users_create_home`: Whether to create home directories (default: `true`)
- `default_user_group`: Default group for users (default: `"users"`)
- `users_home_base_dir`: Base directory for user homes (default: `"/home"`)

For detailed configuration options, refer to the [role documentation](https://github.com/iamenr0s/ansible-role-users).

---

#### ansible-role-ssh-hardening

**Source**: [https://github.com/iamenr0s/ansible-role-ssh-hardening](https://github.com/iamenr0s/ansible-role-ssh-hardening)  
**Location**: `roles/ansible-role-ssh-hardening/`

Hardens the OpenSSH daemon (and optionally the system-wide SSH client) with security best practices. Renders a hardened `sshd_config` from strong cryptographic defaults, removes weak host keys, deploys a pre-login banner, and validates the configuration before restart.

**Key Features**:
- Strong cryptography only: curated cipher, MAC, and key exchange algorithm lists
- Authentication hardening: root login, password auth, empty passwords, Kerberos/GSSAPI all disabled by default
- Access control via `ssh_allow_users`/`ssh_deny_users`/`ssh_allow_groups`/`ssh_deny_groups` and `Match` blocks
- Removes weak DSA host keys and regenerates missing host keys
- Deploys a configurable pre-login banner
- Validates the rendered config with `sshd -t` before and after restart
- Backs up the original `sshd_config` before overwriting it
- Optionally hardens the system-wide SSH client (`/etc/ssh/ssh_config`)

**Example Usage**:
```yaml
- hosts: all
  become: true
  roles:
    - role: ansible-role-ssh-hardening
```

---

#### ansible-role-kernel-configuration

**Source**: [https://github.com/iamenr0s/ansible-role-kernel-configuration](https://github.com/iamenr0s/ansible-role-kernel-configuration)  
**Location**: `roles/ansible-role-kernel-configuration/`

Applies sensible kernel settings via `sysctl` tailored to each supported OS major version, and manages kernel modules (load, blacklist, options). Container-aware: sysctl work is skipped automatically inside containers.

**Key Features**:
- Applies kernel parameters using `sysctl`, keyed by distro and major version
- Persists settings in `kernel_parameters_sysctl_file` and optionally reloads
- Skips sysctl work automatically inside containers (Docker/Podman) — safe to run against both real hosts and containerized CI
- Manages kernel modules: load/unload, persistent loading, blacklisting, and per-module options

**Example Usage**:
```yaml
- hosts: all
  become: true
  roles:
    - role: ansible-role-kernel-configuration
```

---

#### ansible-role-cgroup

**Source**: [https://github.com/iamenr0s/ansible-role-cgroup](https://github.com/iamenr0s/ansible-role-cgroup)  
**Location**: `roles/ansible-role-cgroup/`

Manages Linux cgroup configuration across Debian/Ubuntu and RHEL-family (AlmaLinux/RockyLinux/Fedora) servers. Ensures persistent kernel parameters via GRUB updates and reboots when changes are applied.

**Key Features**:
- Enforces cgroup v2 or v1 consistently
- Adds required kernel parameters idempotently
- Regenerates GRUB on supported distros
- Skips GRUB regeneration/reboot automatically inside containers
- Reboots when changes require it (and when the memory cgroup controller is expected but absent)

**Example Usage**:
```yaml
- hosts: all
  become: true
  roles:
    - role: ansible-role-cgroup
```

---

## Requirements

- Ansible >= 2.9 (some roles require >= 2.13 — see individual role docs)
- Python >= 3.6
- Target systems: Linux distributions (AlmaLinux, Rocky Linux, Fedora, Debian, Ubuntu)

## Dependencies

Collection-level dependencies (declared in `galaxy.yml`):

- `ansible.posix` >= 1.0.0
- `community.general` >= 1.0.0

Some individual roles (e.g. `ansible-role-firewalld`) additionally depend on:

- `iamenr0s.ansible_role_pkg_management`
- `iamenr0s.ansible_role_kernel_configuration`

These are also vendored as submodules under `roles/`, so they are available locally if all submodules are initialized.

## Security Considerations

When using the user management role:

1. **Use Ansible Vault** for sensitive data like SSH private keys and passwords
2. **Encrypt vault files** before committing to version control
3. **Set appropriate file permissions** for vault password files (600)
4. **Review sudo permissions** carefully before deployment
5. **Use strong passwords** and consider password expiration policies

When using `ansible-role-ssh-hardening`:

1. **Test in a non-production environment first** — incorrect hardening can lock you out
2. **Keep a console/KVM fallback** in case SSH access is lost
3. **Verify public-key authentication works** before disabling password auth

## Example Playbook

```yaml
---
- name: Configure Linux systems
  hosts: linux_servers
  become: yes
  vars_files:
    - vars/vault.yml  # Encrypted with ansible-vault

  pre_tasks:
    - name: Install required dependencies
      ansible.builtin.package:
        name:
          - python3
          - sudo
        state: present

  roles:
    - role: ansible-role-etc-hosts
    - role: ansible-role-users
      users_groups:
        - name: developers
        - name: operators
          gid: 2000
      users:
        - name: deploy_user
          comment: "Deployment User"
          group: developers
          manage_ssh_key: true
          sudo_options: "ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp"
    - role: ansible-role-ssh-hardening
    - role: ansible-role-firewalld
```

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the
local pipeline commands and pull request checklist. This project follows the
[Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).

## Security

See [SECURITY.md](SECURITY.md) — GitHub private vulnerability reporting, no
public issues for security bugs.

## License

This project is licensed under the [MIT License](LICENSE).

## Author Information

Author: iamenr0s