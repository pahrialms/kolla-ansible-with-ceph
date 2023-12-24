# kolla-ansible-with-ceph
This is documentation how to installing OpenStack for production with storage distribution Ceph

### Physical Topology
![image](https://github.com/pahrialms/kolla-ansible-with-ceph/assets/82088448/55bc2e43-5a1f-4a04-9289-5ca6685d4e98)

### Bonding architecture
![image](https://github.com/pahrialms/kolla-ansible-with-ceph/assets/82088448/5fb0e1c3-9b69-4ae1-845f-96357505170a)

### Network Cofiguration :

| Host              | BondM          | Bond0.10        | Bond0.30        | Bond0.40        | Bond0.50        | Bond1.20        | Bond1          |
|-------------------|----------------|-----------------|-----------------|-----------------|-----------------|-----------------|----------------|
|                   | Management     | External        | Internal        | Overlay         | Storage-Cluster | Storage-Access  | FIP Network    |
| ms-controller001 | 172.18.2.11/24 | 172.20.20.11/24 | 172.20.21.11/24 | 172.20.22.11/24 | -               | 172.16.16.11/24 |
| ms-controller002 | 172.18.2.12/24 | 172.20.20.12/24 | 172.20.21.12/24 | 172.20.22.12/24 | -               | 172.16.16.12/24 |
| ms-controller003 | 172.18.2.13/24 | 172.20.20.13/24 | 172.20.21.13/24 | 172.20.22.13/24 | -               | 172.16.16.13/24 |
| ms-compute001     | 172.18.2.14/24 | 172.20.20.14/24 | 172.20.21.14/24 | 172.20.22.14/24 | 172.20.23.14/24 | 172.16.16.14/24 |
| ms-compute002     | 172.18.2.15/24 | 172.20.20.15/24 | 172.20.21.15/24 | 172.20.22.15/24 | 172.20.23.15/24 | 172.16.16.15/24 |
| ms-compute003     | 172.18.2.16/24 | 172.20.20.16/24 | 172.20.21.16/24 | 172.20.22.16/24 | 172.20.23.16/24 | 172.16.16.16/24 |

### add domain mapping at /etc/hosts in each node 
```
nano /etc/hosts
---
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
---
```

Install Openstack with kolla-ansible with storage distribution Ceph

1. [Install ceph with cephadm](https://github.com/pahrialms/kolla-ansible-with-ceph/blob/main/ceph/cephadm.md)
2. [Install Openstack with kolla-ansible](https://github.com/pahrialms/kolla-ansible-with-ceph/blob/main/openstack/kolla-ansible.md)
3. [Create resource Openstack](https://github.com/pahrialms/integrate-openstack-kube/blob/main/openstack/create-resource-openstack.md)

