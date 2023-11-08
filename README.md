# CML Example VXLAN Lab Automation
This demo uses the Ansible CML and DCNM collections to automate the deployment of a sample VXLAN EVPN fabric to CML, onboard switches to NDFC, and deploy networking to allow two end hosts to communicate through the fabric.

## Topology
<p align="center">
<img src=https://i.imgur.com/6gzwfjG.png alt= "" width="60%" height="60%">
</p>

## Files
### CML
`cml.yaml` takes the example topology file `topology.yaml`, replaces some values and deploys it to CML
```
- name: Upload lab
  cisco.cml.cml_lab:
    host: "{{ cml_host }}"
    user: "{{ cml_username }}"
    password: "{{ cml_password }}"
    lab: "{{ cml_lab }}"
    file: topology.yaml
    state: started
    // options are absent, present, started, stopped, wiped
```
### NDFC
`onboard.yaml` inventories the CML devices to a NDFC VXLAN EVPN Fabric
```
- name: Add switches to fabric
  cisco.dcnm.dcnm_inventory:
    fabric: "{{ fabric }}"
    state: merged
    config:
    - seed_ip: 10.0.223.185
      auth_proto: MD5
      user_name: "{{ switch_username }}
      password: "{{ switch_password }}"
      max_hops: 0
      role: border_gateway_spine
      preserve_config: false
    [...]
```
`networking.yaml` creates a VRF, two networks, attaches them to interfaces Ethernet1/3 on leaf-103 and leaf-104, and changes those interfaces to access ports. This allows our hosts in CML, labeled 192.168.1.150 and 192.168.2.150, to communicate through the fabric. 

## Installation
| Package     | Version |
| ----------- | ------- |
| NDFC        | 12.1.3b |
| Ansible     | 2.13.1  |
| cisco.cml   | 1.1.0   |
| cisco.dcnm  | 3.4.2   |

```
ansible-galaxy install cisco.dcnm --ignore-certs
```
```
ansible-galaxy install cisco.cml --ignore-certs
```
Ansible CML Collection requires virl2_client:
```
pip3 install virl2_client
```

## Usage
Fabric needs to be created in NDFC prior to running these scripts
### `inventory.yaml`
- Update `ansible_host`, `ansible_user`, `ansible_password` 
### `vars.yaml`
- Update hostname, usernames, passwords
- Change `fabric` in `vars.yaml` to match fabric name on NDFC
### To use your own IPs (`vars.yaml`):
- Update device IPs (lines 12-21)
- Update gateway in networks (lines 31-45)
- Update interfaces (lines 47-53)


To deploy both the CML and NDFC configuration including networking, use
```
ansible-playbook -i inventory.yaml main.yaml
```
To deploy only the CML configuration, use
```
ansible-playbook -i inventory.yaml cml/cml.yaml
```
To only inventory the devices to NDFC, use
```
ansible-playbook -i inventory.yaml ndfc/onboard.yaml
```
To only deploy the networking config to NDFC, use
```
ansible-playbook -i inventory.yaml ndfc/networking.yaml
```
To remove lab from CML, delete networking config and switches from NDFC, use
```
ansible-playbook -i inventory.yaml cleanup.yaml
```
