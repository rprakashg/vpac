Role Name
=========

This role enable virtualization on RHEL host

Requirements
------------

* Red Hat enterprise linux installed and registered on the virtualization host

Host must meet following requirements
* Architecture of host machine supports KVM virtualization
* 6GB free diskspace for the host, plus additional 6GB for each virtual machine.
* 2GB memory for the host, plus another 6GB for each virtual machine
* 4 CPUs for the host. VMs can generally run with a single assigned vCPU, but Red Hat recommends assigning 2 or more vCPUs per virtual machine to avoid VMs becoming unresponsive during high load

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
