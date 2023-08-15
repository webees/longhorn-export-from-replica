https://longhorn.io/docs/1.4.2/advanced-resources/data-recovery/export-from-replica/

https://longhorn.io/kb/troubleshooting-volume-with-multipath/

https://github.com/longhorn/longhorn/wiki/Performance-Benchmark

```shell
curl https://releases.rancher.com/install-docker/20.10.sh | sh

curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.4.2/scripts/environment_check.sh | bash
```

```shell
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras

sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

# https://github.com/longhorn/longhorn/issues/3207
```
# "There were many similar errors. Turn up verbosity to see them." err="orphaned pod \"4778d900-4bec-40ac-95ce-7578e6345677\" found, but error occurred when trying to remove the volumes dir: not a directory" numErrs=3
/var/lib/kubelet/pods/$pod_id/volumes/kubernetes.io~csi/pvc_$pvc_id/

while true ; do
for o in $(tail /var/log/syslog | grep -o  -E 'orphaned pod \\"((\w|-)+)\\' | cut -d" " -f3 | grep -oE '(\w|-)+' | uniq); do
    echo "orphaned pod: $o"
	p="/var/lib/kubelet/pods/$o/volumes/kubernetes.io~csi"
	if [ -d "$p" ] ; then
	  echo "Removing $o"
	  rm -rf "$p"
	fi
done
done
sleep 2
done
```

```shell
#!/bin/bash
apt install -y jq
echo "---------------------------"
docker stop $(docker ps -aq)

umount -R /home

find "/home" -mindepth 1 -maxdepth 1 -type d -name "pvc-*" -exec umount {} \;
find "/home" -mindepth 1 -maxdepth 1 -type d -name "pvc-*" -exec rm -rf {} \;

multipath_conf="/etc/multipath.conf"

if [ ! -f "$multipath_conf" ]; then
    touch "$multipath_conf"
    echo "Created $multipath_conf."
fi

if grep -qF "blacklist" "$multipath_conf"; then
    echo "Required content already exists in $multipath_conf."
else
    echo -e "blacklist {\n devnode "^sd[a-z0-9]+" \n}" >> "$multipath_conf"
    echo "Added required content to $multipath_conf."
fi

systemctl restart multipathd.service

for path in "/var/lib/longhorn/replicas/"*/; do
    echo "---------------------------"
    dir=$(basename "$path")
    meta="$path/volume.meta"
    size=$(jq '.Size' "$meta")
    echo $dir $size
    cat "$meta"
    echo "docker run -d --rm --name $dir -v /dev:/host/dev -v /proc:/host/proc -v /var/lib/longhorn/replicas/$dir:/volume --privileged longhornio/longhorn-engine:v1.4.2 launch-simple-longhorn $dir $size"
          docker run -d --rm --name $dir -v /dev:/host/dev -v /proc:/host/proc -v /var/lib/longhorn/replicas/$dir:/volume --privileged longhornio/longhorn-engine:v1.4.2 launch-simple-longhorn $dir $size
    dev="/dev/longhorn/$dir"
    pvc="/home/$dir"
    mkdir $pvc
    while ! [ -b $dev ]; do
        sleep 1
    done
    mount -o ro $dev $pvc
done

echo "---------------------------"
docker ps
```

# [WARN]  multipathd is running

```shell
#!/bin/bash
apt install -y jq
echo "---------------------------"

multipath_conf="/etc/multipath.conf"

if [ ! -f "$multipath_conf" ]; then
    touch "$multipath_conf"
    echo "Created $multipath_conf."
fi

if grep -qF "blacklist" "$multipath_conf"; then
    echo "Required content already exists in $multipath_conf."
else
    echo -e "blacklist {\n devnode "^sd[a-z0-9]+" \n}" >> "$multipath_conf"
    echo "Added required content to $multipath_conf."
fi

systemctl restart multipathd.service
```
