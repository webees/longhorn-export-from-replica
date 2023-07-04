https://longhorn.io/docs/1.4.2/advanced-resources/data-recovery/export-from-replica/

https://longhorn.io/kb/troubleshooting-volume-with-multipath/

```shell
#!/bin/bash

multipath_conf="/etc/multipath.conf"
content="blacklist {\n    devnode \"^sd[a-z0-9]+\"\n}"

if [ ! -f "$multipath_conf" ]; then
    touch "$multipath_conf"
    echo "Created $multipath_conf."
fi

if grep -qF "blacklist" "$multipath_conf"; then
    echo "Required content already exists in $multipath_conf."
else
    echo -e "$content" >> "$multipath_conf"
    systemctl restart multipathd.service
    echo "Added required content to $multipath_conf."
fi
```
