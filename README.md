# Ansible Linux Collection

A collection of Ansible roles and modules for Linux system administration.

## Description

This collection provides tools and roles for managing Linux systems, including user management, system configuration, and automation tasks.

## Installation

### Install the Collection

```bash
ansible-galaxy collection install community.linux
```

### Initialize Git Submodules

This collection includes external roles as git submodules. After cloning the collection, initialize the submodules:

```bash
git submodule update --init --recursive
```

## Included Roles

### Included Roles (via git submodules)

#### ansible-role-users

**Source**: [https://github.com/iamenr0s/ansible-role-users](https://github.com/iamenr0s/ansible-role-users)  
**Location**: `roles/ansible-role-users/` (git submodule)

This role manages the addition of users and groups to your system. It provides comprehensive user management capabilities including:

- Creating and managing users and groups
- SSH key management
- Sudo configuration
- Password management with encryption
- System user creation
- User removal and cleanup

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

## Requirements

- Ansible >= 2.9
- Python >= 3.6
- Target systems: Linux distributions (Ubuntu, CentOS, RHEL, Debian, etc.)

## Dependencies

- `ansible.posix` >= 1.0.0
- `community.general` >= 1.0.0

## Security Considerations

When using the user management role:

1. **Use Ansible Vault** for sensitive data like SSH private keys and passwords
2. **Encrypt vault files** before committing to version control
3. **Set appropriate file permissions** for vault password files (600)
4. **Review sudo permissions** carefully before deployment
5. **Use strong passwords** and consider password expiration policies

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
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

GPL-2.0-or-later

## Author Information

Maintained by the community. For issues and contributions, please visit the [GitHub repository](https://github.com/yourusername/ansible-linux-collection).

## Support

For issues related to:
- **This collection**: Open an issue in this repository
- **The users role**: Visit [ansible-role-users repository](https://github.com/iamenr0s/ansible-role-users)