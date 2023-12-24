## 1. Install package dependency for Openstack
```
# run only on node deployer (aka controller001)
apt-get install python3-dev libffi-dev gcc libssl-dev python3-selinux python3-setuptools python3-venv -y
```

## 2. Create virtaul environment
```
# run only on node deployer
mkdir openstack
cd ~/openstack
python3 -m venv os-venv
source os-venv/bin/activate
```

## 3. Upgrade pip, install ansible & install kolla-ansible
```
# run only on node deployer
pip install -U pip
pip install 'ansible>=6,<8'
pip install git+https://opendev.org/openstack/kolla-ansible@master
```

## 4. Install ansible galaxy 
```
# run only on node deployer
kolla-ansible install-deps
```
## 5. Copy openstack configuration from kolla directory
```
# run only on node deployer
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla

cp -r os-venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp os-venv/share/kolla-ansible/ansible/inventory/* .
```

## 6. Create ansible configuration tune
```
# run only on node deployer
mkdir -p /etc/ansible
nano /etc/ansible/ansible.cfg
---
[defaults]
host_key_checking=False
pipelining=True
forks=100
---
```

## 7. Edit multinode file 
```
# run only on node deployer
nano multinode
---
[control]

ms-controller001
ms-controller002
ms-controller003

[network:children]
compute

[compute]
ms-compute001
ms-compute002
ms-compute003

[monitoring]
ms-controller001
ms-controller002
ms-controller003

[storage]
ms-controller001
ms-controller002
ms-controller003

[deployment]
localhost       ansible_connection=local
```

## 8. Test connectivity kolla-ansible to all node
```
ansible -i multinode all -m ping
```

## 9. Generate password for openstack services
```
kolla-genpwd
```

## 10. Edit kolla globals.yml
```
nano /etc/kolla/globals.yml
---
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "master"
kolla_internal_vip_address: "172.20.21.100"
kolla_internal_fqdn: "internal.xyz.local"
kolla_external_vip_address: "172.20.20.100"
kolla_external_fqdn: "public.xyz.local"
neutron_external_interface: "bond1"
enable_neutron_provider_networks: "yes"
enable_haproxy: "yes"
enable_keystone: "yes"
enable_horizon: "yes"
enable_openstack_core: "yes"
enable_cinder: "yes"
enable_cinder_backup: "no"
enable_fluentd: "no"
enable_masakari: "yes"
enable_masakari_instancemonitor: "yes"
enable_masakari_hostmonitor: "yes"
enable_hacluster: "yes"
external_ceph_cephx_enabled: "yes"
cinder_backend_ceph: "yes"
ceph_cinder_keyring: "ceph.client.cinder.keyring"
ceph_cinder_user: "cinder"
ceph_cinder_pool_name: "volumes"
glance_backend_ceph: "yes"
ceph_glance_keyring: "ceph.client.glance.keyring"
ceph_glance_user: "glance"
ceph_glance_pool_name: "images"
nova_backend_ceph: "yes"
ceph_nova_keyring: "ceph.client.cinder.keyring"
ceph_nova_user: "cinder"
ceph_nova_pool_name: "vms"
kolla_enable_tls_internal: "yes"
kolla_enable_tls_external: "yes"
kolla_copy_ca_into_containers: "yes"
kolla_enable_tls_backend: "yes"
openstack_cacert: "/etc/ssl/certs/ca-certificates.crt"
kolla_external_vip_interface: "bond0.10"
api_interface: "bond0.30"
tunnel_interface: "bond0.40"
neutron_plugin_agent: "openvswitch"
enable_neutron_dvr: "yes"
---
```

## 11. Copy ceph keyring to kolla config directory
```
# run only on node deployer
mkdir -p /etc/kolla/config/cinder/cinder-volume/
mkdir /etc/kolla/config/nova/
mkdir /etc/kolla/config/glance/

cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/cinder/cinder-volume/
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/nova/
cp /etc/ceph/ceph.client.glance.keyring /etc/kolla/config/glance/

* note : remove tab space in ceph.conf 

cp /etc/ceph/ceph.conf /etc/kolla/config/cinder/
cp /etc/ceph/ceph.conf /etc/kolla/config/glance/
cp /etc/ceph/ceph.conf /etc/kolla/config/nova/
```

## 12. Execute deployment command 
```
# run only on node deployer
kolla-ansible -i ./multinode certificates
kolla-ansible -i ./multinode bootstrap-servers
kolla-ansible -i ./multinode prechecks
kolla-ansible -i ./multinode deploy

kolla-ansible -i ./multinode post-deploy

pip3 install openstackclient

echo "export OS_CACERT=/etc/ssl/certs/ca-certificates.crt" | tee -a /etc/kolla/admin-openrc.sh
cat /etc/kolla/certificates/ca/root.crt | sudo tee -a /etc/ssl/certs/ca-certificates.crt
```

## 14. To support vlan for provider network, change neutron-server config to add network_vlan_ranges
```
# change config on all controller
vim /etc/kolla/neutron-server/ml2_conf.ini
---
[ml2_type_vlan]
network_vlan_ranges = physnet1:100:1110
---
# restart neutron_server on all controller
docker restart neutron_server
```


