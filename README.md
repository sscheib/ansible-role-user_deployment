[![ansible-lint](https://github.com/sscheib/ansible-role-user_deployment/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/sscheib/ansible-role-user_deployment/actions/workflows/ansible-lint.yml) [![Publish latest release to Ansible Galaxy](https://github.com/sscheib/ansible-role-user_deployment/actions/workflows/ansible-galaxy.yml/badge.svg)](https://github.com/sscheib/ansible-role-user_deployment/actions/workflows/ansible-galaxy.yml)

user_deployment
=========
This is a *very* simple role that will perform user and group deployment, and add SSH keys to the users.

Role Variables
--------------
| variable                                     | default                      | required | description                                                                    |
| :---------------------------------           | :--------------------------- | :------- | :----------------------------------------------------------------------------- |
| `udm_users`                                  | unset                        | true     | Users to deploy                                                                |
| `udm_enable_passwordless_sudo`               | `false`                      | false    | Whether to enable passwordless sudo for the privileged group (e.g. `wheel`)    |
| `udm_quiet_assert`                           | `false`                      | false    | Whether to quiet the assert statements                                         |

## Variable `udm_users`

An extended example of only the `udm_users` variable is illustrated down below:
```
udm_users:
  udm_enable_passwordless_sudo: true
  udm_quiet_assert: false
  udm_users:
    - name: 'steffen'                             # Name of the user
      groups:                                     # List of groups to create (if not existent) and add the user to
        - name: 'mygroup'                         # Name of the group
          gid: 2001                               # Group ID for the group
      comment: 'My personal user'                 # Comment to add to the user (aka gecos)
      uid: 1000                                   # User ID for the user
      home: '/home/steffen'                       # Path to the home directory
      create_home: true                           # Whether to create a home directory
      authorized_keys:                            # List of authorized keys. Either by specifying the path to a file or specifying the SSH keys inline as string
        - '/home/steffen/.ssh/id_ecdsa.pub'       # Path to an existent SSH key on the *local* machine
        - !vault |                                # Inline specified SSH key
          $ANSIBLE_VAULT;1.1;AES256
      password: !vault |                          # Password of the user. If not specified, it will be set to '*', which is a passwordless account
        $ANSIBLE_VAULT;1.1;AES256
      privileged: true                            # Whether this user is a privileged user and should be added to the respective group (e.g. RHEL: wheel, Debian: sudo)
      remove_unspecified_ssh_keys: true           # Whether to remove unspecified SSH keys from the user's authorized_keys file
      always_update_password: false               # Whether to always update the password (which is not idempotent), or if it only should be set when creating a user
```
The only required option for a user is the `name`. Everything else can be mixed and matched.

Dependencies
------------

This role makes use of the [Ansible Posix collection](https://github.com/ansible-collections/ansible.posix).

Example Playbook
----------------

```
---
- hosts: 'all'
  gather_facts: false
  roles:
    - role: 'user_deployment'
      vars:
        udm_enable_passwordless_sudo: true
        udm_quiet_assert: false
        udm_users:
          - name: 'steffen'
            groups:
              - name: 'mygroup'
                gid: 2001
            comment: 'steffen'
            uid: 1000
            home: '/home/steffen'
            create_home: true
            authorized_keys:
              - '/home/steffen/.ssh/id_ecdsa.pub'
              - !vault |
                $ANSIBLE_VAULT;1.1;AES256
            password: !vault |
                $ANSIBLE_VAULT;1.1;AES256
            privileged: true                                                                                                                                                   
            remove_unspecified_ssh_keys: true
            always_update_password: false
...
```

License
-------

GPL-2.0-or-later
