# Ansible Hands on Technical Sales Workshop

## Ansible for Network Automation
This lab material is designed to augment the instructor lead Red Hat Ansible Tower two-day hands on workshop with additional exercises that reinforce the general Ansible concepts with labs for Ansible Netorking.

### GitHub / GitLab Account
If you do not currently have a [GitHub](github.com) and [GitLab](gitlab.com) account, please create a personal (free) account on each platform.

#### Intro to Git for Network Engineers
When Implementing Infrastructure as Code concepts, the Source of Truth for work-flow, credentials, and configuration data will be stored centrally in a Source (Version) Control System. Refer to these resources:

 * https://www.slideshare.net/joelwking/introduction-to-git-for-network-engineers-lab-guide
 * https://www.slideshare.net/joelwking/introduction-to-git-for-network-engineers

The above links provide a working knowledge of Git to complete the remaining training excercises.

### Chrome Postman
Download and install Chrome Postman at https://www.getpostman.com/. 

You can verify the installation by following the training material at https://www.getpostman.com/docs/v6/postman/launching_postman/sending_the_first_request.

When you feel comfortable that you have Postman installed and can issue an API call to the *docs.postman-echo.com/* server, make an API call to the APIC controller using the following collection.

#### Postman collection
This [collection](https://gist.github.com/joelwking/a819d1f00748122eaeeb3eda7edf9c10) can be used as an example on how to use Chrome Postman for basic API calls to an ACI fabric. After installing Postman, download and import the collection.

### Labs
These labs make use of networking devices including APIC controllers on ACI fabrics, Arista (EoS), Cisco IOS routers and Nexus switches.

#### Create copies of the training repositories
A fork is a copy of a repository. Forking a repository allows you to create a copy under your account as a starting point for your labs.

Fork these repositories from your GitHub account:

 * https://github.com/joelwking/ansible-aci
 * https://github.com/joelwking/ansible-cse-training

#### ACI Ansible Sandbox
This is an on-demand lab in the ATC [portal](atcportal.apps.wwt.com).

Portal link: https://atcportal.apps.wwt.com/#/practices/network/capability/2aafd0b14f031f005ec5e57d0210c712
Lab guide: http://labs.wwtlab.net/lab-guides/aci-automation-lab-guide/welcome.html

This lab provides both an ACI virtual simulator and Ansible Tower instance within the sandbox. 

##### Cisco ACI Sandbox
This lab teaches the concept of how the `hosts:`, inventory and `hostname` variable work together to execute a playbook on the control node (Ansible Tower) to configure an APIC controller.

For this exercise, you will modify the playbook and inventory to use the Cisco ACI sandbox instead of the ACI simulator in the on-demand lab. Refer to: https://developer.cisco.com/site/aci/. The Cisco ACI sandbox environment, https://sandboxapicdc.cisco.com, can be accessed from the Ansible Tower instance in the on-demand lab.

Modify `aci_tenant_demo.yml` to use the variable `inventory_hostname` in the `aci_tenant` module. Modify the `hosts:` variable in the playbook to reference a group name you create called `ciscosandbox`. Create the group in your inventory with the proper host. Update your credentials accordingly.


##### Create and delete an EPG in the demo fabric
Using the playbook `aci_tenant_demo.yml` as a guide, create a new playbook to Manage End Point Groups (EPG) objects (fv:AEPg)

References:
 * Cisco ACI Guide https://docs.ansible.com/ansible/latest/scenario_guides/guide_aci.html
 * Network modules: ACI https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#aci

The playbook will first need to create a Bridge Domain and Application Profile. Pass the value of 'present' or 'absent' to the playbook as an extra variable so the playbook can optionally create or delete these objects.

#### Credentials
The file `playbooks/passwords.yml` is a clear text file where we manage our usernames and passwords for the network devices in the lab. Use `ansible-vault` to encrypt this file and replace the clear text file with the encrypted file. On your Ansible Tower instance, you will need to create a *vault* credential with the secret you used to encrypt the `playbooks/passwords.yml` in order for Ansible Tower to decrypt the file when you include the file into your playbooks.


#### Inventory
Managing the inventory source is a fundamental component of using Ansible for network automation. For Ansible Engine, inventory sources can be static or dynamic. In the past, inventory files were in INI format, now YAML is supported and the preferred method. The file, `inventory/inventory.yml` defines the devices available for the following exercises.

The playbook `ping.yml` provides exercises to understand how to manipulate your inventory source and run a playbook to verify your inventory using the *ping* command from the control node.

Use the sample `inventory/inventory.yml` file and with the Ansible Tower GUI, create the equivalent inventory in your Ansible Tower instance.

#### Gathering facts from network devices

One common use case for network automation is to identify access ports and enable a configuration to support Identity Services Engine (ISE) dot1x. As part of this workflow, the switch IOS version will also need to be identified and possibly upgraded to support an ISE implementation.

Another use case is to verify that a port on a network switch is not in use. The playbook `verify_port.yml` is a sample playbook which determines the network operating system of a switch based on the inventory file, and then includes the appropriate task file to gather facts from the switch. It then demonstrates how to use the `assert` command to test conditionals against the structured data returned from the switch.

After becoming familiar with this playbook, create a new playbook which optionally update the IOS version of the switch if required to support ISE. Reload the switch to enable the upgrade, skip the remaining steps in the playbook.  If the IOS version running supports ISE, update the access ports on a switch or group of switches configuration with the port commands to enable ISE. 

The execution of this playbook will be to run in two passes, first to upgrade the IOS, the second pass to update the port configuration after the switch has rebooted to the desired version.

For information on ISE switch port configuration, do a web search for `ise switch port configuration`.

#### Declarative and imperative configuration management

This playbook, `switch_interface.yml`, contrasts declarative and imperative configuration management of a Nexus (NX-OS) switch interface. In the imperative mode, the playbook uses the Jinja template module to create a configuration file from a template.  It also demonstrates using the `lookup` plugin is used to read a file into a playbook variable. 

Use the module `nxos_banner` along with a Jinja template, to create a custom MOTD banner which includs a variable passed to the playbook from a Ansible Tower survey or extra var.

#### JunOS netconf

*THIS SECTION INTENTIONALLY LEFT BLANK*

####  F5 Programmability/Automation Training Class

The goal of this lab is to introduce and enable F5 administrators in taking the first step to automate, simplify, and streamline the F5 provisioning and configuration process. This is an on-demand lab in the ATC [portal](atcportal.apps.wwt.com)

[F5 Automation and Programmability Lab](https://atcportal.apps.wwt.com/#/myatc/mycapabilities/08b3ac9f4fb12a400bc050ee0310c7a6)


## Resources
Included are additional links and resources for network automation and programmability.

### WWT Lab guides
  * [F5 Automation Lab Guide](http://labs.wwtlab.net/automation-lab/)
  * [ACI Automation Lab Guide](http://labs.wwtlab.net/lab-guides/aci-automation-lab-guide/)

### LinkLight
Training Course for Ansible Network Automation
  * https://network-automation.github.io/linklight/

### GitHub organizations and content
  * https://github.com/network-automation
  * https://github.com/datacenter
  * https://github.com/gzapodea/DevNet_Create_2018
  * https://github.com/gzapodea/BRKNMS_2935_Orlando
  * https://github.com/hpreston

### DevNet and CiscoLive 2018 Orlando
  * https://developer.cisco.com/
  * [Micro-Service Applications for Infrastructure People](https://clnv.s3.amazonaws.com/2018/usa/pdf/BRKCLD-1009.pdf)
  * [From Zero to Network Programmability in 120 minutes – DNA Center, RESTCONF, NETCONF, WebEx Teams and ServiceNow](https://clnv.s3.amazonaws.com/2017/usa/pdf/BRKRST-2935.pdf)

## Author
Joel W. King (GitHub / GitLab @joelwking) joel.king@wwt.com