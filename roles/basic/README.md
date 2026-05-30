Role Name
=========

This Ansible role is designed for the initial configuration of instances, primarily for installing basic packages, creating directories, changing the hostname and performing basic tasks such as configuring the firewall.

Requirements
------------

* Ansible 2.9+ installed on the control machine.
* SSH access to all Swarm nodes with proper credentials.

Role Variables
--------------

The role provides the following customizable variables. These should be defined in the playbook or inventory:
| Variable       | Default Value  | Description    |
|----------------|----------------|----------------|
| timezone       | "Asia/Tashkent"| Timezone       |
| update_repo    | true           | Update repo cache |
| selinux_disable| true           | Disable SELinux in system |
| additional_packages | htop,mc, etc | Packages which should be installed on the system |

Dependencies
------------
None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: karen.basic }

License
-------

This role is licensed under the MIT License.

Author Information
------------------

Abbos Pulatov from AnyOps teams
