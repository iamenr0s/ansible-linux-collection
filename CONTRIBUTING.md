# Contributing

Thanks for taking the time to contribute to `iamenr0s.ansible_linux_collection`!

This is an Ansible collection containing multiple roles vendored as git submodules from `iamenr0s/ansible-role-*` repositories. Individual role logic lives in those upstream repos; this collection aggregates them.

## Collection Structure

```
.
├── galaxy.yml              # Collection metadata
├── README.md               # Collection documentation
├── LICENSE                 # MIT
├── .gitmodules            # Submodule registrations
├── roles/
│   ├── ansible-role-upgrade/
│   ├── ansible-role-swap/
│   ├── ansible-role-cri-o/
│   ├── ansible-role-etc-hosts/
│   ├── ansible-role-firewalld/
│   ├── ansible-role-containerd/
│   ├── ansible-role-users/
│   ├── ansible-role-ssh-hardening/
│   ├── ansible-role-kernel-configuration/
│   └── ansible-role-cgroup/
└── docs/
    └── users-role-guide.md
```

## Getting Started

1. Fork the repository and create your branch from `main`.
2. Initialize the git submodules:

   ```bash
   git submodule update --init --recursive
   ```

3. Install development dependencies:

   ```bash
   pip3 install ansible ansible-lint yamllint
   ansible-galaxy collection install -r requirements.yml  # if present
   ```

## Making Changes

- Keep changes small and focused — one topic per pull request.
- Follow the existing YAML style; lint rules live in `.yamllint` and `.ansible-lint`.
- If you add or change a variable, document it in `README.md` and set a sane default in `defaults/main.yml` of the relevant role.
- All roles require `become: yes` (privilege escalation) unless explicitly documented otherwise.

## Testing Individual Roles

Each role can be tested independently using Molecule. Refer to the individual role's `CONTRIBUTING.md` for role-specific testing instructions:

- [ansible-role-upgrade CONTRIBUTING](https://github.com/iamenr0s/ansible-role-upgrade/blob/main/CONTRIBUTING.md)
- [ansible-role-users CONTRIBUTING](https://github.com/iamenr0s/ansible-role-users/blob/main/CONTRIBUTING.md)
- (other roles follow the same pattern)

To run lint checks on the collection:

```bash
yamllint .
ansible-lint
```

## Submitting a Pull Request

1. Ensure lint checks pass locally.
2. Fill in the pull request template.
3. A maintainer will review your PR.

## Reporting Bugs and Requesting Features

Use the issue templates — they ask for the details (distro, Ansible version, role variables) needed to reproduce a problem.

## Where to Make Changes

| Change Type | Where to Submit |
|-------------|-----------------|
| Bug fix / new feature for a specific role | Submit to the [upstream role repo](https://github.com/iamenr0s) |
| Collection-level changes (README, galaxy.yml, bundling) | Submit PR to this collection |
| Documentation improvements | Submit to this collection |

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating you agree to abide by it.
