nextcloud_davfs
===============
[![Ansible Lint](https://github.com/oxivanisher/role-nextcloud_davfs/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/oxivanisher/role-nextcloud_davfs/actions/workflows/ansible-lint.yml)

Install and configure your nextcloud client directory on a linux server with the help of davfs.

Requirements
------------

You need a Nextcloud (ownCloud will probably also work). Also you should use a service password to connect to the cloud.

Role Variables
--------------

| Name                                    | Comment                                                                   | Default value          |
| --------------------------------------- | ------------------------------------------------------------------------- | ---------------------- |
| nextcloud_davfs_osuser                  | The OS user of the directory will be configured in                        |                        |
| nextcloud_davfs_osgroup                 | The OS group of the directory will be configured in                       | nextcloud_davfs_osuser |
| nextcloud_davfs_osdir                   | The directory in the os users home where the cloud will be mounted        | `Nextcloud`            |
| nextcloud_davfs_cloud_url               | Your Nextcloud webdav URL (i.e. https://your.cloud.tld/remote.php/webdav) |                        |
| nextcloud_davfs_cloud_user              | Your Nextcloud user                                                       |                        |
| nextcloud_davfs_cloud_password          | Your Nextcloud password                                                   |                        |
| nextcloud_davfs_nextrsync               | Enable `nextrsync` feature                                                | `false`                |
| nextcloud_davfs_nextrsync_tmpfs         | Put the `nextrsync` directory in a tmpfs                                  | `true`                 |
| nextcloud_davfs_nextrsync_dirs          | List of Nextcloud subdirectories to sync into `~/.nextrsync`              | `[scripts]`            |
| nextcloud_davfs_nextrsync_post_commands | List of shell commands to run after each sync                             | `[]`                   |
| nextcloud_davfs_nextrsync_interval      | How often the systemd timer fires (systemd time span syntax)              | `5min`                 |
| nextcloud_davfs_nextrsync_on_boot_delay | Delay before the first run after boot (systemd time span syntax)          | `1min`                 |
| nextcloud_davfs_cache_size              | The davfs cache size                                                      | `500`                  |

Dependencies
------------

The `nextrsync` functionality synchronizes configured Nextcloud subdirectories into `~/.nextrsync` (optionally on a tmpfs) so you can add it to `$PATH` without waiting for davfs on every access.

When enabled, the role:
- Deploys `/usr/local/bin/nextrsync` from a template (configured via `nextcloud_davfs_nextrsync_dirs` and `nextcloud_davfs_nextrsync_post_commands`)
- Creates a systemd service that runs as the configured user and depends on the davfs mount unit, so it only runs when the mount is active
- Creates a systemd timer that triggers the service on boot and periodically thereafter

Example configuration:
```yaml
nextcloud_davfs_nextrsync: true
nextcloud_davfs_nextrsync_dirs:
  - Documents/something_important
  - scripts
nextcloud_davfs_nextrsync_post_commands:
  - "python3 /usr/local/bin/make_scripts_executable.py /home/myuser/.nextrsync/scripts"
```

Example Playbook
----------------
```yaml
- name: Configure Nextcloud devfs
  hosts: jump
  collections:
    - oxivanisher.linux_base
  roles:
    - role: oxivanisher.linux_base.nextcloud_davfs
```

License
-------

BSD

Author Information
------------------

This role is part of the [oxivanisher.linux_base](https://galaxy.ansible.com/ui/repo/published/oxivanisher/linux_base/) collection, and the source for that is located on [github](https://github.com/oxivanisher/collection-linux_base).
