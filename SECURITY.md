# Security Policy

## Supported Versions

Only the latest released version of this collection receives security fixes.

| Version | Supported          |
| ------- | ------------------ |
| Latest  | :white_check_mark: |
| Older   | :x:                |

## Reporting a Vulnerability

Please **do not** open a public issue for security vulnerabilities.

Instead, report them privately via [GitHub private vulnerability reporting](https://github.com/iamenr0s/ansible-linux-collection/security/advisories/new).

Include a description of the issue, steps to reproduce, and the affected role/distro/Ansible version if relevant.

You can expect an initial response within 7 days. Once the issue is confirmed, a fix will be released as soon as practical and you will be credited in the release notes unless you prefer otherwise.

## Security Best Practices

When using this collection:

1. **Use Ansible Vault** for sensitive data (passwords, SSH keys, vault files)
2. **Encrypt vault files** before committing to version control
3. **Set appropriate file permissions** for vault password files (600)
4. **Test in non-production first** — especially for SSH hardening and firewall roles
5. **Keep a console/KVM fallback** in case SSH hardening locks you out
