configure_bridge_interfaces
=========

This role configures linux bridge interfaces on the target host machines

Requirements
------------
Ansible version 2.14 and above

Role Variables
--------------

Role access a list of bridge interfaces to be created.

| Variable Name | Purpose | Optional |
| ------------- | ------- | -------- |
| bridge_interfaces | List of bridge interfaces to be created | false (role would just simply return with nothing to do) |

Dependencies
------------

This role leverages [community.general.nmcli](https://docs.ansible.com/projects/ansible/latest/collections/community/general/nmcli_module.html) collection to create the required bridge interfaces on target host machines

Example Playbook
----------------

Including an example of how to use this role:

    - hosts: virtual_hosts
      become: true
      vars:
        bridge_interfaces:
        - name:  station-bus
          nic: ens5f0
          ip4: 10.107.159.1/24
          gw4: 10.107.159.254
          dns4: 10.107.1.54

      roles:
      - rprakashg.vpac.configure_bridge_interfaces

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
