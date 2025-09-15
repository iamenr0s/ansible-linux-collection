# User Management Role Guide

This guide provides detailed information about using the `iamenr0s.ansible_role_users` role integrated into this collection.

## Overview

The ansible-role-users role provides comprehensive user and group management capabilities for Linux systems. It's integrated into this collection as a git submodule and offers features like:

- User and group creation/deletion
- SSH key management
- Sudo configuration
- Password management with encryption
- Home directory management

## Installation

This role is included as a git submodule. After cloning the collection, initialize the submodules:

```bash
git submodule update --init --recursive
```

## Basic Usage

### Creating Users and Groups

```yaml
- name: Basic user management
  hosts: all
  become: yes
  roles:
    - role: ansible-role-users
      users_groups:
        - name: developers
          gid: 1000
        - name: admins
          gid: 1001
      users:
        - name: alice
          comment: "Alice Developer"
          uid: 1000
          group: developers
          manage_ssh_key: true
        - name: bob
          comment: "Bob Admin"
          uid: 1001
          group: admins
          groups:
            - developers
```

### Advanced SSH Key Management

```yaml
- name: Advanced SSH configuration
  hosts: all
  become: yes
  roles:
    - role: ansible-role-users
      users:
        - name: deploy_user
          manage_ssh_key: true
          copy_private_key: true  # Generate and copy private key
          authorized_keys:
            - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB... user1@workstation"
            - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB... user2@laptop"
        - name: service_account
          private_keys:
            - name: id_rsa
              content: |
                -----BEGIN OPENSSH PRIVATE KEY-----
                b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAAB...
                -----END OPENSSH PRIVATE KEY-----
```

### Sudo Configuration

```yaml
- name: Configure sudo access
  hosts: all
  become: yes
  roles:
    - role: ansible-role-users
      users:
        - name: admin_user
          sudo_options: "ALL=(ALL) NOPASSWD: ALL"
        - name: service_user
          sudo_options:
            - "ALL= NOPASSWD: /usr/bin/systemctl restart nginx"
            - "ALL= NOPASSWD: /usr/bin/systemctl reload nginx"
            - "ALL= NOPASSWD: /usr/bin/systemctl status nginx"
```

### Password Management

```yaml
- name: Set user passwords
  hosts: all
  become: yes
  roles:
    - role: ansible-role-users
      users:
        - name: user_with_password
          # Generate hash with: python3 -c "import crypt; print(crypt.crypt('password', crypt.mksalt(crypt.METHOD_SHA512)))"
          password: "$6$rounds=656000$salt$hash..."
          update_password: on_create  # Only set password on user creation
          password_validity_days: 90
          expires: -1  # Account never expires
```

## Security Best Practices

### Using Ansible Vault

1. **Create a vault file for sensitive data:**

```bash
ansible-vault create vars/vault.yml
```

2. **Store sensitive information in the vault:**

```yaml
# vars/vault.yml
vault_ssh_private_key: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  ...
  -----END OPENSSH PRIVATE KEY-----

vault_user_password: "$6$rounds=656000$..."
```

3. **Reference vault variables in your playbook:**

```yaml
- name: Secure user management
  hosts: all
  become: yes
  vars_files:
    - vars/vault.yml
  roles:
    - role: ansible-role-users
      users:
        - name: secure_user
          password: "{{ vault_user_password }}"
          private_keys:
            - name: id_rsa
              content: "{{ vault_ssh_private_key }}"
```

### Vault Password Management

```bash
# Create vault password file
echo "your_vault_password" > .vault-password
chmod 600 .vault-password

# Configure ansible.cfg
cat << EOF > ansible.cfg
[defaults]
vault_password_file = ./.vault-password
EOF
```

## Common Patterns

### Development Environment Setup

```yaml
- name: Setup development environment
  hosts: dev_servers
  become: yes
  roles:
    - role: ansible-role-users
      users_groups:
        - name: developers
        - name: docker
          system: true
      users:
        - name: "{{ item }}"
          comment: "Developer - {{ item }}"
          group: developers
          groups:
            - docker
            - sudo
          manage_ssh_key: true
          sudo_options: "ALL=(ALL) NOPASSWD: ALL"
      loop:
        - alice
        - bob
        - charlie
```

### Production Server Hardening

```yaml
- name: Production server user setup
  hosts: prod_servers
  become: yes
  roles:
    - role: ansible-role-users
      users_groups:
        - name: app_users
          gid: 2000
        - name: monitoring
          gid: 2001
          system: true
      users:
        - name: app_deploy
          uid: 2000
          group: app_users
          manage_ssh_key: true
          sudo_options:
            - "ALL= NOPASSWD: /usr/bin/systemctl restart myapp"
            - "ALL= NOPASSWD: /usr/bin/systemctl status myapp"
          password_validity_days: 30
        - name: monitoring_user
          uid: 2001
          group: monitoring
          system: true
          manage_ssh_key: false
```

## Troubleshooting

### Common Issues

1. **Permission denied errors:**
   - Ensure the playbook runs with `become: yes`
   - Check that the target user has sudo privileges

2. **SSH key issues:**
   - Verify SSH key format and permissions
   - Check that `.ssh` directory has correct permissions (700)

3. **Group membership problems:**
   - Ensure groups exist before assigning users to them
   - Check group creation order in your playbook

### Debugging

Enable verbose output to troubleshoot issues:

```bash
ansible-playbook -vvv your-playbook.yml
```

## Role Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `default_user_shell` | `/bin/bash` | Default shell for users |
| `users_create_home` | `true` | Create home directories |
| `users_sudo_includedir` | `"#includedir /etc/sudoers.d"` | Sudo include directory |
| `default_user_group` | `"users"` | Default group for users |
| `users_home_base_dir` | `"/home"` | Base directory for user homes |

## Links

- [Original Role Repository](https://github.com/iamenr0s/ansible-role-users)
- [Ansible User Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
- [Ansible Group Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group_module.html)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html)