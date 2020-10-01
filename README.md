# F5 Automation Toolchain demo for Ansible
## Overview

The F5 Automation Toolchain is a declarative approach to deploy BIG-IPs configuration over a REST based API. This based on three different extensions:

- Declarative Onboarding (DO)
- Application Service 3 (AS3)
- Telemetric Streaming (TS)

I will show in this demo, how you can deploy the AS3 extension over ansible on a BIG-IP, and how you can do declarative deployments over ansible based on AS3.

## Requirements
For this demo I assume that you installed already Ansible on your system. To use the F5 modules on ansible, it is necessary to install additional the following extensions via pip:

```
pip install bigsuds
pip install f5-sdk
```

I tested the Demo with the following Versions:

```
python --version
#> Python 2.7.9
ansible --version
#> ansible 2.7.7
pip show f5-sdk
#> Name: f5-sdk
#> Version: 3.0.20
```

## Install the demo

If you got git available, the easiest way would be to clone it:

```
git clone https://github.com/rabru/ansible_f5AT.git
```

An alternative way would be to download the zip file and unpack it on the target system.

## Install f5 collection

You need to go into the demo folder and install the collection:

```
cd ansible_f5AT
ansible-galaxy collection install f5networks.f5_modules
```

To validate the installed version use:

```
ansible-galaxy collection list
```

## Configuration Setup
Next you need to adapt the settings regarding your environment.

### Adapt the hosts file

You need to adapt the host setting at the hosts file. The easiest way would be to adapt the IP towards the mgmt IP of your target system.

It is highly recommended to put the passwords into an ansible vault. To keep this demo as simple as possible I put it into the hosts file.

```
cat hosts
# output:
[bigip]
10.10.86.22 username="admin" password="admin" server_port="443"

[bigip-ha]
10.10.86.30 username="admin" password="admin" server_port="443"
10.10.86.31 username="admin" password="admin" server_port="443"

```

## Application Services 3 (AS3)
Here you can find the [AS3 documentation.](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/)

### Install AS3 Extension

Before we can install the AS3 extension at the BIG-IP, we need to download the install package. You can find the extension [here](https://github.com/F5Networks/f5-appsvcs-extension/releases). 

Be aware, that the path may change over time:
```
cd playbooks/files/

wget https://github.com/F5Networks/f5-appsvcs-extension/releases/download/v3.22.1/f5-appsvcs-3.22.1-1.noarch.rpm
```

Please adapt the AS3Version variable at the playbook/AS3_install.yaml playbook.

Next you can install the package on the target BIG-IP. Please make sure, that rpm is available on your ansible host!:

```
ansible-playbook playbooks/AS3_install.yaml -e target=bigip -e state=present
```

To remove the extension from the BIG-IP, you need to change the state to absent:

```
ansible-playbook playbooks/AS3_install.yaml -e target=bigip -e state=absent
```

## File based Ansible integration over REST

In this example of an AS3 deployment, we simply use a json file with the complete configuration, which we will send over REST towards the target system. Here you need to adapt the related json file, which requires some basic AS3 knowledge:

```
cat playbooks/files/http_simple.json
#output:
{
   "class": "AS3",
   "action": "deploy",
   "persist": true,
   "declaration": {
      "class": "ADC",
      "schemaVersion": "3.0.0",
      "id": "urn:uuid:33045210-3ab8-4636-9b2a-c98d22ab915d",
      "label": "Sample 1",
      "remark": "Simple HTTP application with RR pool",
      "Sample_01": {
         "class": "Tenant",
         "A1": {
            "class": "Application",
            "template": "http",
            "serviceMain": {
               "class": "Service_HTTP",
               "virtualAddresses": [
                  "10.128.10.101"
               ],
               "pool": "web_pool"
            },
            "web_pool": {
               "class": "Pool",
               "monitors": [
                  "http"
               ],
               "members": [{
                  "servicePort": 80,
                  "serverAddresses": [
                     "10.10.10.210",
                     "10.10.10.211"
                  ]
               }]
            }
         }
      }
   }
}

```

Additional you should have a look at the json file for the remove request:
```
cat playbooks/files/remove_http_simple.json
# output:
{
   "class": "AS3",
   "action": "deploy",
   "persist": true,
   "declaration": {
      "class": "ADC",
      "schemaVersion": "3.0.0",
      "id": "urn:uuid:33045210-3ab8-4636-9b2a-c98d22ab915d",
      "label": "Sample 1",
      "remark": "Simple HTTP application with RR pool",
      "Sample_01": {
         "class": "Tenant"
      }
   }
}

```

You can start the deployment with the following command line:

```
ansible-playbook playbooks/AS3_http_simple_REST.yaml -e target=bigip -e state=present
```

And remove the configuration like this:

```
ansible-playbook playbooks/AS3_http_simple_REST.yaml -e target=bigip -e state=absent
```


## Template based Ansible integration over REST

This integration based on a [Jinja2](http://jinja.pocoo.org/) template, which enable you to abstract some settings into the playbook. The main advantage of this method is, that the DevOps user doesn't need to understand the AS3 json syntax. it is enough to adapt the variable in the playbook:
```
cat playbooks/AS3_http_simple_temp_REST.yaml | head -n20
# output:
---
- name: AS3 http setup
  hosts: "{{ target }}"
  gather_facts: false
  vars:
    tenant: "myTenant"
    appName: "httpApp"
    vip: "10.128.10.102"
    memberAddr:
      - "10.10.10.203"
      - "10.10.10.204"

  tasks:

  - name: get auth token
    uri:
      url: https://{{ inventory_hostname }}/mgmt/shared/authn/login
      method: POST
```

As you can see, it is much easier to adapt the playbook to the needs of the infrastructure.

Deployment:
```
ansible-playbook playbooks/AS3_http_simple_temp_REST.yaml -e target=bigip -e state=present
```

Remove:
```
ansible-playbook playbooks/AS3_http_simple_temp_REST.yaml -e target=bigip -e state=absent
```

## Template based Ansible integration over REST with multiply Applications

This integration I did based on rules. I will document this later.

