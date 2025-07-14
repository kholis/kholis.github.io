---
{"dg-publish":true,"permalink":"/resources/it/linux/bash-notes/","tags":["#awk","#delimiter","#separator","bash"],"created":"2024-07-05T08:47:50.824+07:00","updated":"2025-07-14T18:00:55.038+07:00"}
---

# Coloring PS1
Add this line to `~/.bashrc`
```bash
export PS1="\[\033[36;1m\]\u\[\033[m\]@\[\033[32;1m\]\h:\[\033[33;1m\]\w\[\033[m\]\$ "
```

Result:
![Pasted image 20250714141432.png](/img/user/Resources/IT/Linux/Pasted%20image%2020250714141432.png)

Bash color code: https://gkarthiks.github.io/quick-commands-cheat-sheet/bash_command.html

# Get script BASE_DIR
## 1. from claude.ai
```bash
#!/bin/bash

# Get the directory of the script
get_script_dir() {
    # Get the source directory
    SOURCE="${BASH_SOURCE[0]}"

    # Resolve $SOURCE until the file is no longer a symlink
    while [ -h "$SOURCE" ]; do
        DIR="$( cd -P "$( dirname "$SOURCE" )" && pwd )"
        SOURCE="$(readlink "$SOURCE")"
        # If $SOURCE was a relative symlink, we need to resolve it 
        # relative to the path where the symlink file was located
        [[ $SOURCE != /* ]] && SOURCE="$DIR/$SOURCE"
    done

    # Get the canonical path of the script directory
    DIR="$( cd -P "$( dirname "$SOURCE" )" && pwd )"

    echo "$DIR"
}

# Call the function and store the result
SCRIPT_DIR=$(get_script_dir)

# Print the result
echo "The script is located in: $SCRIPT_DIR"

# You can use $SCRIPT_DIR for further operations in your script
# For example:
# cd "$SCRIPT_DIR"
# ... (rest of your script)
```

## 2. stackoverflow
```bash
#!/usr/bin/env bash

SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )
```

is a useful one-liner which will give you the full directory name of the script no matter where it is being called from.

It will work as long as the last component of the path used to find the script is not a symlink (directory links are OK). If you also want to resolve any links to the script itself, you need a multi-line solution:

```bash
#!/usr/bin/env bash

SOURCE=${BASH_SOURCE[0]}
while [ -L "$SOURCE" ]; do # resolve $SOURCE until the file is no longer a symlink
  DIR=$( cd -P "$( dirname "$SOURCE" )" >/dev/null 2>&1 && pwd )
  SOURCE=$(readlink "$SOURCE")
  [[ $SOURCE != /* ]] && SOURCE=$DIR/$SOURCE # if $SOURCE was a relative symlink, we need to resolve it relative to the path where the symlink file was located
done
DIR=$( cd -P "$( dirname "$SOURCE" )" >/dev/null 2>&1 && pwd )
```

This last one will work with any combination of aliases, `source`, `bash -c`, symlinks, etc.
Ref: https://stackoverflow.com/a/246128/1377004

# Get filename and extension
ref: https://stackoverflow.com/a/965072/1377004
```bash
filename=$(basename -- "$fullfile")
extension="${filename##*.}"
filename="${filename%.*}"
```

# Replace file extension
```
find . -iname "*.txt" -exec bash -c 'mv "$0" "${0%\.txt}.md"' {} \;
```
ref: https://www.reddit.com/r/ObsidianMD/comments/qgyjij/comment/kux05ai/

# Awk OFS delimiter separator
`-F`: input delimiter
`OFS`: output delimiter
```
awk -F "\t" -v OFS="|" '{print $1, $2, $3}'
```

# Find file only on current dir then compress
```
$ find /path/ -maxdepth 1 -type f -exec tar czf /tmp/file.tar.gz {} \+ 
```
ref: https://stackoverflow.com/a/16703837/1377004

# sshfs with id_rsa
```
sshfs -o allow_other,defer_permissions,IdentityFile=~/.ssh/id_rsa root@xxx.xxx.xxx.xxx:/ /mnt/droplet
```
ref: https://vtcri.kayako.com/article/86-sshfs-to-mount-remote-file-systems-over-ssh

in fstab:
```
sshfs#USER@domain.com:/data/www /mnt/logs/  fuse IdentityFile=/home/USER/.ssh/id_rsa,uid=UID,gid=GUID,users,idmap=user,noatime,allow_other,_netdev,reconnect,ro 0 0
```
ref: https://ivan.reallusiondesign.com/mount-sshfs-volumes-in-fstab-with-ssh-key/

What you need to do is specify which private key to use in the `~/.ssh/config` file. for example:

```
Host server1.nixcraft.com
    IdentityFile ~/backups/.ssh/id_dsa
Host server2.nixcraft.com
    IdentityFile /backup/home/userName/.ssh/id_rsa
```
ref: https://unix.stackexchange.com/a/61571/55247

# Check return value
`&&` execute when return value 0
`||` execute when return value not 0
ex:
```bash
nc -z host $PORT && echo port $PORT open || echo port $PORT close
```

# Retry until succeed
```bash
MAX_RETRIES=10
RETRY_DELAY=60  # seconds
COUNT=0

while true; do
    ./your-script.sh
    if [ $? -eq 0 ]; then
        echo "Script succeeded."
        break
    fi
    COUNT=$((COUNT+1))
    if [ $COUNT -ge $MAX_RETRIES ]; then
        echo "Script failed after $MAX_RETRIES attempts."
        exit 1
    fi
    echo "Retrying in $RETRY_DELAY seconds..."
    sleep $RETRY_DELAY
done
```