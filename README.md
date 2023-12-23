# kolla-ansible-with-ceph
This is documentation how to installing OpenStack for production with storage distribution Ceph


![image](https://github.com/pahrialms/kolla-ansible-with-ceph/assets/82088448/55bc2e43-5a1f-4a04-9289-5ca6685d4e98)

![image](https://github.com/pahrialms/kolla-ansible-with-ceph/assets/82088448/7d19bbb8-299b-4840-b36d-ed35d12e974f)

Network Cofiguration :


| Host  | bondm | bond0.10| bond0.30 | bond0.40| bond0.50 | bond1.20/| bond1 FIP |
|       |  management | external api | internal api | overlay | storage cluster | storage access | |
| ----- | --------- | ---- | ------ | ----- | ---- | ---- | ------ |
| ms-controller001 | 172.127 |
|

