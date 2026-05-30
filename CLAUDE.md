# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A standalone Ansible role development lab. Roles here are developed and tested locally before being promoted to `../ansible/`. The default target host is `vm-os` at `192.168.56.101` (VirtualBox VM, user `ahmad`).

## Common Commands

```bash
# Run a single-role playbook against the VM
ansible-playbook -i hosts.yml playbooks/nginx.yml

# Run with a specific tag
ansible-playbook -i hosts.yml playbooks/nginx.yml --tags=config

# Full DR bootstrap (all roles in order)
ansible-playbook -i hosts.yml playbooks/bootstrap.yml

# Ad-hoc verification
ansible -i hosts.yml all -m ansible.builtin.shell -a 'nginx -t'
ansible -i hosts.yml all -m ansible.builtin.shell -a 'systemctl status nginx'

# Molecule: full test lifecycle (requires Docker)
cd roles/<role_name>
molecule test

# Molecule: apply only (faster iteration)
molecule converge

# Molecule: verify only
molecule verify

# Override distro for docker-install role (default: rockylinux9)
MOLECULE_DISTRO=rockylinux9 molecule test
```

All roles have Molecule tests. Roles with `prepare.yml` (`ssh`, `compose-deploy`) install prerequisites before converge runs.

## Role Inventory

| Role | Molecule platforms | Key variables |
|------|-------------------|--------------|
| `basic` | ubuntu-2404, rocky-10 | `timezone`, `additional_packages` |
| `users` | ubuntu-2404, rocky-10 | `users`, `user_groups`, `users_to_keep` |
| `ssh` | ubuntu-2404, rocky-10 | `ssh_port`, `ssh_extra_allow_users`, `users` |
| `zsh` | ubuntu-2404, rocky-10 | `zsh_users`, `zsh_theme` |
| `nginx` | ubuntu-2404, rocky-10 | `nginx_sites`, `nginx_allowlist_ips` |
| `haproxy` | ubuntu-2404, rocky-10 | `haproxy_http_services`, `haproxy_tcp_services` |
| `security` | ubuntu-2404, rocky-10 | `sysctl_hardening`, `security_disabled_services` |
| `docker-install` | rocky-9 (MOLECULE_DISTRO) | — |
| `compose-deploy` | ubuntu-2404 (privileged) | `SERVICE_NAME`, `IMAGE`, `HARBOR_ENABLED` |
| `devops_packages` | ubuntu-2404, rocky-10 | `binaries`, `cli_aliases` |

## Architecture Notes

**nginx role — site object shape:** Sites defined in `group_vars/all/nginx.yml` under `nginx_sites`. Each entry supports `listen_ssl`, `use_allowlist` (references `nginx_allowlist_ips`), `log_format` (`main` or `json`), and a `locations` list with optional `proxy_pass` and raw `extra` block. Template at `roles/nginx/templates/site.conf.j2`.

**users role — server targeting:** Each user entry has a `servers` field. The role creates users that match `inventory_hostname`, match any `group_names` entry, or are set to `['all']`. Users in `/home` not in `users` + `users_to_keep` are removed (home directory kept). SSH keys are stored in `files/ssh_keys/` and referenced as `files/ssh_keys/<name>` in user definitions.

**ssh role — AllowUsers:** Builds the `AllowUsers` directive from the `users` variable (same format as the users role) plus `ssh_extra_allow_users`. Designed to run alongside the users role.

**haproxy role — conf.d pattern:** Main config at `/etc/haproxy/haproxy.cfg` includes `/etc/haproxy/conf.d/*.cfg`. HTTP services, TCP services, and stats get separate fragments. Config is validated with `haproxy -c` before reload.

**compose-deploy role:** Generates `docker-compose.yaml` via Jinja2, deploys to Docker Swarm. Harbor login is skipped when `HARBOR_ENABLED: false` or image URL doesn't contain `harbor.imv.uz`. Molecule test uses `privileged: true` with `prepare.yml` to install Docker and init Swarm.

**devops_packages role — binary installs:** Fetches latest GitHub release assets matching `asset_pattern` regex and copies to `binary_install_dir`. Molecule converge limits to `yq` only to keep test time short.

**bootstrap.yml:** Applies all roles in dependency order (basic → users → ssh → security → docker → packages → zsh). Use this as the DR playbook for rebuilding a server from scratch.

## Molecule Notes

- `zsh` molecule requires internet access (downloads oh-my-zsh and plugins via git/curl).
- `devops_packages` molecule requires internet access (downloads binaries from GitHub releases).
- `compose-deploy` molecule runs ubuntu-2404 only (Docker-in-Docker complexity).
- `ssh` molecule includes `prepare.yml` to install and start openssh-server before converge.
