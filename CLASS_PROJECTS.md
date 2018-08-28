# Class Projects
As a means to apply the concepts learning in the formal training exercises along with the sample playbooks provided in this repository, the following represents typical use cases where Ansible could be applied to solve common requirements.

## Cisco ISE Implementation

[Cisco Identity Services Engine (ISE)](https://www.cisco.com/c/en/us/products/security/identity-services-engine/index.html) provides a means to dynamically controlling network access. ISE deployments in a campus environment often require software upgrades to Catalyst switches and extensive configuration changes to both the global and interface configurations.

The following is a functional description of a real-world requirement for implementing ISE on thousands of Catalyst switches.

### Functional Description
The following sections provide a functional description of the sources and sinks (input and output) of data, and the functional requirements for the processing which must be automated for this use case.

#### Discovery
In this network, discovery has been effected and a CSV file providing an inventory of devices is available. The format of the file is:

```
switch_name, management_ip
```
Assume a *service account* can be obtained with the appropriate credentials and privilege level 15 enabling configuration changes.

##### Determine switch OS

To determine if the switch OS is at the desired version, create a data structure as follows:


```yaml
---
#
# filename: /files/switch_series.yml
#
    series:
        C3850: 
          filename: cat3k_caa-universalk9.SPA.03.06.06.E.152-2.E6.bin
          version: "03.06.06E"
          models:
            - WS-C3850-12S
            - WS-C3850-24T
        C3560:
          filename: foo
          version: "03.06.06E"
          models:
            - bar
        C3750:
        C4500-X: 
        C2950: 
        C3750-X: 
        C2960-XR: 
        C3550: 
        C2960: 
        C3560-X: 
        C2960-X: 
        C3650: 
        C4900: 
        C3500-XL: 
        C2960-S:

```
Note: the above series numbers must be verified. The series is derived from `ansible_net_model`. 

Use `ios_facts` to return the facts of the device, for example:

```json
        "ansible_net_model": "WS-C3850-24T",
        "ansible_net_serialnum": "FOC1731X1LR",
        "ansible_net_stacked_models": [
            "WS-C3850-24T"
        ],
        "ansible_net_stacked_serialnums": [
            "FOC1731X1LR"
        ],
        "ansible_net_version": "03.06.06E"
```

You can then split the model into `[u'WS', u'C3850', u'24T']` and reference the second list element `[1]` to reference the appropriate switch series from above. 
```
"{{ mongo.ansible_facts.ansible_net_model.split('-')[1] }}"
```
By comparing the desired version with the running version on the switch, use module `add_host` to update the inventory with the switches which require an upgrade.

##### Layer 3 devices
Switches which are configured as Layer 3 devices may have multiple IP addresses. If the device has a Loopback0 interface, capture the IP address of the Loopback0 interface and use this IP address when creating the import (CSV) file to load into the ISE GUI.

Example of testing if a Loopback0 is defined on the switch.

```yaml
   when:  mongo.ansible_facts.ansible_net_interfaces['Loopback0'] is defined
```   
Example of JSON output from module ios_facts.
```json
            "ansible_net_interfaces": {
                  "Loopback0": {
                        "bandwidth": 8000000,
                        "description": "THIS DEVICE HAS A LOOPBACK",
                        "duplex": null,
                        "ipv4": [
                            {
                                "address": "192.0.2.1",
                                "subnet": "31"
                            }
                        ],
                        "lineprotocol": "up ",
                        "macaddress": null,
                        "mediatype": null,
                        "mtu": 1514,
                        "operstatus": "up",
                        "type": null
                    }
            }
```
###### Secondary IP addresses
Loopback interfaces may be configured with secondary IP addresses, providing more than one array element in the "ipv4" variable. The structured data returned from the switch does specify the primary (or secondary) IP address when more than
one address is configured. In testing, the configured secondary IP address is the first array element, with the primary IP addressing being the second (last) array element. 

If more than one IP address exists on the Loopback interface, flag these devices for manual remediation.

#### IOS Upgrade
Create an Ansible role which upgrades a switch to the desired OS level. The role should download the appropriate file to the switch. Combine the switch_series.yml input file with a variable definition of the location of the file:

```yaml
    image:
        server: "ftp://{{ ftp.username }}:{{ ftp.password }}@10.255.40.101/IOS/images/ios/"

```
Using `ios_command`, download the image to the `flash:` file system on the switch. Optionally schedule a reboot
`reload at 12:00 1 june 2019 rebooted by Ansible`   at the input maintenance window. 

Note: If date and time are not configured, the `reload` command will fail: `%The date and time must be set first.` Re-mediate and switches in this condition. 

##### Verification

TODO: This section will provide a means to verify the switch(s) have rebooted and are running the desired network OS.

#### ISE Configuration

TODO: This section will use Jinja templates to generate the desired configuration to implement the desired ISE configuration.

##### Verification

TODO: This section will verify the switch has been configured for ISE.

#### Analysis

##### Errors
When testing using `ansible 2.6.0` we encountered the error referencing variables from `ios_facts` 
['rawunicodeescape' codec can't decode bytes in position 4090-4093: truncated \uXXXX](https://github.com/ansible/ansible/issues/43748).  Upgrade to `ansible 2.6.3` resolved the problem.

##### Sample playbook to gather facts from a Catalyst 3850 switch

```yaml
#!/usr/bin/ansible-playbook
---
#
#      Copyright (c) 2018 World Wide Technology, Inc.
#      All rights reserved.
#
#      author: Joel W. King,  World Wide Technology
#
#      usage:
#          ./ios_xe_structured.data.yml -v -i ./inventory/class/inventory.yml
#          ./ios_xe_structured.data.yml -v --skip-tags=mongo -i ./inventory/class/inventory.yml
#
#      inventory:
#
#          cat3850:
#            hosts:
#              cat3850-asset86597.sandbox.wwtatc.local: {}
#            vars:
#              ansible_connection: network_cli
#              ansible_network_os: ios
#              ansible_become: yes
#              ansible_become_method: enable
#              ansible_ssh_user: admin

#
- name:   Catalyst (IOS-XE) example playbook
  hosts:  cat3850
  gather_facts: yes

  vars_files:
    - "{{ playbook_dir }}/passwords.yml"
  vars:
    iso8601: "{{ansible_date_time.iso8601}}"
    control_host: "{{ansible_hostname}}"
    ansible_ssh_pass: "{{ ios.password }}"

  tasks:
    - name: Use ios_facts to query the facts about the device
      ios_facts:
        gather_subset: all
      register: mongo

    - name: Create the document
      set_fact:
        config_data: "{{ {'iso8601': iso8601, 'control_host': control_host,  'config': mongo.ansible_facts } }}"

    - debug:
        msg: "{{ mongo.ansible_facts.ansible_net_interfaces['Loopback0'] }}"
      when:  mongo.ansible_facts.ansible_net_interfaces['Loopback0'] is defined

```
## Author
Joel W. King (joel.king@wwt.com)