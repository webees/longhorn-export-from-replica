https://longhorn.io/docs/1.4.2/advanced-resources/data-recovery/export-from-replica/

https://longhorn.io/kb/troubleshooting-volume-with-multipath/

```shell
curl https://releases.rancher.com/install-docker/20.10.sh | sh
```

```shell
#!/bin/bash

multipath_conf="/etc/multipath.conf"

if [ ! -f "$multipath_conf" ]; then
    touch "$multipath_conf"
    echo "Created $multipath_conf."
fi

if grep -qF "blacklist" "$multipath_conf"; then
    echo "Required content already exists in $multipath_conf."
else
    echo 'blacklist { devnode "^sd[a-z0-9]+" }' >> "$multipath_conf"
    systemctl restart multipathd.service
    echo "Added required content to $multipath_conf."
fi

for path in "/var/lib/longhorn/replicas/"*/; do
    dir=$(basename "$path")
    meta="$path/volume.meta"
    size=$(jq '.Size' "$meta")
    echo $dir $size
    cat "$meta"
    echo "docker run -d --name $dir -v /dev:/host/dev -v /proc:/host/proc -v /var/lib/longhorn/replicas/$dir:/volume --privileged longhornio/longhorn-engine:v1.4.2 launch-simple-longhorn $dir $size"
    docker run -d --name $dir -v /dev:/host/dev -v /proc:/host/proc -v /var/lib/longhorn/replicas/$dir:/volume --privileged longhornio/longhorn-engine:v1.4.2 launch-simple-longhorn $dir $size
    echo "---------------------------"
done

docker ps
```
