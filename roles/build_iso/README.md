build_iso
=========

This role can be used to build a custom ISO with cloud init files for first boot initialization and a custom anaconda kickstart for completely unattended installation.

Requirements
------------

Requires imagebuilder host setup and configured using [setup_imagebuilder](../setup_imagebuilder/) role

Role Variables
--------------

Below you will find description of the settable variables for this role. 

| Variable Name | Purpose |
| ------------- | ------- |
| retries | Number of retries for compose job status |
| delay | delay between retries for compose job status |
| skip_blueprint_creation | Flag to turn off blueprint creation. Useful when you haven't made any changes |
| skip_compose | Skip starting a new compose job, useful when you want to leverage artifacts from an existing compose job. If this is set to true then compose_job_id variable is required. |
| compose_job_id | Compose job id, this is automatically set when compose job is triggered. If compose job is skipped then the compose job id must be passed as value |
| builder_blueprint_name | osbuild blueprint name |
| builder_blueprint_description | osbuild blueprint description |
| builder_blueprint_distro | RHEL distro to use |
| builder_compose_pkgs | additional packages that need to be included |
| builder_compose_customizations | Customizations |
| builder_kickstart_options | Anaconda kickstart options to include |


Dependencies
------------

None

Example Playbook
----------------

Below is a example playbook that demonstrates bulding custom iso images

```yaml
tasks:
  - name: Load secrets from ansible vault
    ansible.builtin.include_vars:
      file: "./vars/secrets.yml"

  - name: Import role
    ansible.builtin.import_role:
      name: rprakashg.vpac.build_iso
```

Full playbook can be found [here](../../playbooks/build_iso.yml)

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
