Cloudbuilder
=========

This is a roles for provisioning clouds nodes/vpcs/vnets.  It takes a model and creates that architecture in the cloud provider specified by that blueprint.

Requirements
------------

These playbooks assume that you have your cloud environment setup correctly for authentication.  For example, boto is required for AWS.

Role Variables
--------------

- `cloud_model`: The cloud model that you want to deploy into the specified cloud provider (required)
- `cloud_project`: The name of the overall project. (DEFAULT: username)
- `cloud_instance`: The specific instance of the overall project. (DEFAULT: `cloud_model`)
- `cloud_provider`: The cloud provider in which to deploy the model. (DEFAULT: aws)
- `cloud_region`: The region of the cloud provider in which to deploy the model (DEFAULT: us-east-1)
- `cloud_cidr` : The private subnet for the deployment in CIDR notation.
- `cloud_site_number`: The site number (DEFAULT: 0)
- `cloud_key_name`: The name of the key used/created (DEFAULT: username)
- `cloud_public_key_file`: The name of the file containing the public key
- `cloud_inventory_root`: The root where the inventories will be created (DEFAULT: `./inventory`)
- `cloud_inventory_dir`: The directory where the inventory files will be created (DEFAULT: `cloud_inventory_root`/`cloud_project`)
- `cloud_inventory_file`: The name of the inventory file (DEFAULT: `cloud_inventory_dir`/`cloud_instance`)


Dependencies
------------

None.

Example Playbook
----------------

    - hosts: localhost
      connection: local
      gather_facts: no
      tasks:
        - assert:
            that:
              - cloud_model is defined
            msg: "You must specify a model, e.g. -e 'cloud_model=csr-spoke1.yml'"

        - include_role:
            name: cloudbuilder
            tasks_from: build
          tags:
            - build

        - include_role:
            name: cloudbuilder
            tasks_from: facts
          tags:
            - facts

        - debug: var=hostvars

        - include_role:
            name: cloudbuilder
            tasks_from: inventory
          tags:
            - inventory

License
-------

GPL-3

Author Information
------------------

- Steven Carter
