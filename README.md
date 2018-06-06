# Cloudbuilder

This is a roles for provisioning clouds nodes/vpcs/vnets.  It takes a model and creates that architecture in the cloud provider specified by that blueprint.

## Requirements

These playbooks assume that you have your cloud environment setup correctly for authentication.  For example, boto is required for AWS.

## Role Variables

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


## Dependencies

None.

## Example Playbook

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

            ## Cloud Models

            The cloud model is a cloud-agnostic data model that describes the particlar cloud deployment.  It creates the container network (e.g. VPC), the subnets, adds routes, security groups, etc.  It also contains instances, but this is mainly to deploy VNFs and service nodes.

            ### Cloud Model Example

                cloud_acls:
                  host-acl:
                    - { src_ip: 0.0.0.0/0, dst_ports: 22, proto: tcp }
                    - { src_ip: 0.0.0.0/0, dst_ports: 443, proto: tcp }
                    - { src_ip: 0.0.0.0/0, proto: icmp }
                  rtr-acl:
                    - { src_ip: 0.0.0.0/0, proto: all }
                vpc_list:
                  - name: "{{ cloud_name }}"
                    provider: "{{ cloud_provider }}"
                    region: "{{ cloud_region }}"
                    project: "{{ cloud_name }}"
                    cidr: "{{ cloud_cidr }}"
                    networks:
                      - name: "outside.{{ cloud_name }}"
                        cidr: "{{ cloud_cidr | ipsubnet(24, 0) }}"
                        az: "{{ cloud_region }}a"
                      - name: "inside.{{ cloud_name }}"
                        cidr: "{{ cloud_cidr | ipsubnet(24, 1) }}"
                        az: "{{ cloud_region }}a"
                        routes:
                          - cidr: "0.0.0.0/0"
                            instance: "rtr1.{{ cloud_name }}"
                            if_index: 2
                    instances:
                      - name: "rtr1.{{ cloud_name }}"
                        size: medium
                        image: csr-byol
                        interfaces:
                          - name: GigabitEthernet1
                            subnet: "outside.{{ cloud_name }}"
                            acl: rtr-acl
                            public_ip: true
                            private_ip: "{{ cloud_cidr | ipsubnet(24, 0) | ipaddr(-2) | ipaddr('address') }}"
                          - name: GigabitEthernet2
                            subnet: "inside.{{ cloud_name }}"
                            acl: rtr-acl
                            public_ip: true
                            private_ip: "{{ cloud_cidr | ipsubnet(24, 1) | ipaddr(-2) | ipaddr('address') }}"
                        tags:
                          network_os: ios
                          groups: network,routers,spoke_routers
                        user_data: "{{ lookup('file', 'csr.user_data') }}"
                      - name: "host1.{{ cloud_name }}"
                        size: micro
                        image: rhel7
                        interfaces:
                          - name: eth0
                            subnet: "inside.{{ cloud_name }}"
                            acl: host-acl
                            private_ip: "{{ cloud_cidr | ipsubnet(24, 2) | ipaddr(10) | ipaddr('address') }}"
                        tags:
                          groups: hosts

## License

GPL-3

## Author Information

* Steven Carter
