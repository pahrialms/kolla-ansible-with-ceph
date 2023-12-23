

Generate SSH-Key and distribute to all host.
```
# run only on controller001
ssh-keygen -t rsa
ssh-copy-id -f -i ~/.ssh/id_rsa.pub root@host
```

Add hostname mapping in /etc/hosts file configuration :
```
# run on all host
cat <<EOF>> /etc/hosts
172.20.20.11 ms-controller001.public.xyz.local
172.20.21.11 ms-controller001 ms-controller001.internal.xyz.local

172.20.20.12 ms-controller002.public.xyz.local
172.20.21.12 ms-controller002 ms-controller002.internal.xyz.local

172.20.20.13 ms-controller003.public.xyz.local
172.20.21.13 ms-controller003 ms-controller003.internal.xyz.local

172.20.20.14 ms-compute001.public.xyz.local
172.20.21.14 ms-compute001 ms-compute001.internal.xyz.local

172.20.20.15 ms-compute002.public.xyz.local
172.20.21.15 ms-compute002 ms-compute002.internal.xyz.local

172.20.20.16 ms-compute003.public.xyz.local
172.20.21.16 ms-compute003 ms-compute003.internal.xyz.local

172.20.20.100 public.xyz.local
172.20.21.100 internal.xyz.local
EOF
```

Install cephadm 
```
# run only on controller001
CEPH_RELEASE=18.2.0 # replace this with the active release
curl --silent --remote-name --location https://download.ceph.com/rpm-${CEPH_RELEASE}/el9/noarch/cephadm

chmod +x cephadm

./cephadm add-repo --release reef
./cephadm install

```

Bootstrap ceph cluster
```
cephadm bootstrap --mon-ip 172.16.16.11 --cluster-network 172.20.23.0/24 --log-to-file --allow-overwrite
```

Install ceph client 
```
cephadm shell
exit
cephadm shell -- ceph -s
cephadm add-repo --release reef
cephadm install ceph-common
```

Copy ceph public key to all node ceph cluster:
```
ssh-copy-id -f -i /etc/ceph/ceph.pub root@controller002
ssh-copy-id -f -i /etc/ceph/ceph.pub root@controller003
ssh-copy-id -f -i /etc/ceph/ceph.pub root@compute001
ssh-copy-id -f -i /etc/ceph/ceph.pub root@compute002
ssh-copy-id -f -i /etc/ceph/ceph.pub root@compute003
```

Add additional ceph-mon
```
ceph orch apply mon --unmanaged
ssh ms-controller002 apt install podman -y
ssh ms-controller003 apt install podman -y

ceph orch host add ms-controller002 172.16.16.12 --labels _admin
ceph orch host add ms-controller003 172.16.16.13 --labels _admin

ceph orch daemon add mon ms-controller002:172.16.16.12
ceph orch daemon add mon ms-controller003:172.16.16.13
```

Add node compute as storage disk server 
```
ceph orch host add ms-compute001 172.16.16.14
ceph orch host add ms-compute002 172.16.16.15

ceph orch device ls
# add disk avaialble as osd
ceph orch daemon add osd *<host>*:*<device-path>*
ex: ceph orch daemon add osd ms-compute001:/dev/sdb
ceph -s
```

Create openstack keyring for openstack usage
```
ceph -s
ceph osd pool create volumes
ceph osd pool create images
ceph osd pool create backups
ceph osd pool create vms

rbd pool init volumes
rbd pool init images
rbd pool init backups
rbd pool init vms

ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images' mgr 'profile rbd pool=images'
ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=volumes, profile rbd pool=vms'

ceph auth get client.glance > /etc/ceph/ceph.client.glance.keyring
ceph auth get client.cinder > /etc/ceph/ceph.client.cinder.keyring

ceph osd lspools
```




