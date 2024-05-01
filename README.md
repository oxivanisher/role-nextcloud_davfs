nextcloud_davfs
===============

Install and configure your nextcloud client directory on a linux server with the help of davfs.
There are two special functions concerning `nextrsync` and `cleanup`.

Requirements
------------

You need a Nextcloud (ownCloud will probably also work). Also you should use a service password to connect to the cloud.

Role Variables
--------------

| Name                           | Comment                                                                   | Default value |
|--------------------------------|---------------------------------------------------------------------------|---------------|
| nextcloud_davfs_osuser         | The OS user the directory will be configured in                           |               |
| nextcloud_davfs_osdir          | The directory in the os users home where the cloud will be mounted        | `Nextcloud`   |
| nextcloud_davfs_cloud_url      | Your Nextcloud webdav URL (i.e. https://your.cloud.tld/remote.php/webdav) |               |
| nextcloud_davfs_cloud_user     | Your Nextcloud user                                                       |               |
| nextcloud_davfs_cloud_password | Your Nextcloud password                                                   |               |
| nextcloud_davfs_nextrsync      | Enable `nextrsync` feature                                                | `False`       |
| nextcloud_davfs_cleanup        | Enable `cleanup` feature                                                  | `False`       |
| nextcloud_devfs_cache_size     | The davfs cache size                                                      | `500`         |

Dependencies
------------

The `nextrsync` functionality synchronizes the configured folders to a tmpfs directory. So that you can add it to your `$PATH` variable without waiting for the davfs to sync all the time. For this to work, you need this bash script in `scripts/nextrsync`:
```bash
#!/bin/bash

# this script should be run very often (i.e. via cron job every 5 minutes)
# it is important to call it from the next/owncloud directory in cron!

# destination folder for the sync
# if you want to change this, please fix also the path in the .bashrc!
DST=$HOME/.nextrsync

# folders to be synced
DIRS="Settings/Linux/ scripts"

RSYNC=$(command -v rsync)
if [ -z "$RSYNC" ];
then
        echo "ERROR: rsync was not found!"
        exit 1
fi

SRC="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
for DIR in $DIRS;
do
        mkdir -p $DST/$DIR
        rsync --delete -a $SRC/../$DIR/ $DST/$DIR
done

touch /tmp/.last_nextrsync_run

$HOME/.nextrsync/scripts/make_scripts_executable.py $HOME/.nextrsync/scripts
```

The `cleanup` functionality removes files older than X days from your cloud and other directories. For this to work, you need this bash script in `scripts/cleanup`:
```bash
#!/bin/bash

#make sure the directory is existing
function checkdir {
        if [ ! -d "$1" -a -d "$(dirname "$1")" ];
        then
                mkdir -p "$1"
        fi
}

#search and remove empty folders
function cleanemptydir {
        find $1 -type d | sort -rn | while read LINE;
        do
                if [ $(find "$LINE" -type f | wc -l) == 0 ];
                then
                        rmdir "$LINE"
                fi
        done

        checkdir "$1"
}

function cleanup {
        checkdir "$1"

        if [ -d "$1" ];
        then
                echo -e "cleaning files in ($1) older than $2 days"
                echo -e "\t`du -sh $1 | awk '{print $1}'` used before"
                #find $1 -maxdepth 1 -not -type d -mtime +$2 -exec rm '{}' \;
                find $1 -not -type d -mtime +$2 -exec rm '{}' \;
                cleanemptydir "$1"
                echo -e "\t`du -sh $1 | awk '{print $1}'` used after"
        else
                echo -e "directory ($1) does not exist."
        fi
}

# ensure the tmp/_not_backuped thing in Nextcloud does not get cleaned
mkdir -p $HOME/Nextcloud/tmp/_not_backed_up
touch $HOME/Nextcloud/tmp/_not_backed_up/.keep
touch $HOME/Nextcloud/tmp/.nomedia

cleanup "$HOME/Downloads/" 30
cleanup "$HOME/tmp/" 30
cleanup "$HOME/nextcloud/tmp/" 90
cleanup "$HOME/Nextcloud/tmp/" 90
cleanup "/media/workdata/tmp/" 90
```

Since the nextrsync folder is hardcoded for the cleanup feature, you can not only enable `cleanup`.

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
