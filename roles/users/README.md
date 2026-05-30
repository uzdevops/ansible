# users Role

## Overview

The `users` role is an Ansible role designed to manage Linux system users efficiently. It enables user creation, deletion, and modification, ensuring consistent user management across multiple servers.

## Features

- Create new users with custom attributes (UID, groups, shell, etc.)
- Delete users and remove their home directories
- Manage user SSH keys for secure authentication
- Ensure users belong to the correct groups
- Set or update user passwords (hashed for security)
- Disable or lock users when necessary

## Requirements

- Ansible 2.9+ (recommended latest stable version)
- Compatible with major Linux distributions (Ubuntu, CentOS, Debian, RHEL, etc.)

## Role Variables

The following variables can be customized in your playbook or inventory:

```yaml
ssh_key_path: files
users:
- name: abbos
  user_groups:
    - 'operations'
    - 'docker'
  servers:
    - 'bus'
    - 'app'
    - 'db'
  ssh_keys:
    - {{ ssh_key_path }}/abbos

user_groups:
  - 'operations'
  - 'docker'
  - 'servers'
  - 'nginx'
```

### Default Variables
```yaml
users: []  # List of users to be managed
remove_users: []  # List of users to be removed
manage_ssh: true  # Enable or disable SSH key management
```

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: karen.users
      vars:
        ssh_key_path: files
        users:
        - name: abbos
          user_groups:
            - 'operations'
            - 'docker'
          servers:
            - 'bus'
            - 'app'
            - 'db'
          ssh_keys:
            - {{ ssh_key_path }}/abbos
```

## Usage

1. Add this role to your Ansible playbook.
2. Define user configurations in `defaults/main.yml` or pass them in your playbook.
3. Run the playbook:
   ```sh
   ansible-playbook -i inventory playbook.yml
   ```

## License

This project is licensed under the **MIT License** – see the [LICENSE](LICENSE) file for details.

## Author

Maintained by Abbos Pulatov from AnyOps team. Contributions are welcome!

