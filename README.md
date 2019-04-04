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
# python --version
Python 2.7.9
# ansible --version
ansible 2.7.7
# pip show f5-sdk
Name: f5-sdk
Version: 3.0.20
```

## Install the demo

If you got git available, the easiest way would be to clone it:

```
# git clone git@github.com:rabru/ansible_f5AT.git
```

An alternative way would be to download the zip file and unpack it on the target system.

## Configuration Setup
Next you need to adapt the settings to your environment.

### Adapt the hosts file

You need to adapt the host setting at the hosts file. The easiest way would be to adapt the IP towards the mgmt IP of your target system.

It is highly recommended to put the passwords into an ansible vault. To keep this demo as simple as possible I put it into the hosts file.

```
# cat hosts
[bigip]
10.10.86.22 username="admin" password="admin"

[bigip-ha]
10.10.86.30 username="admin" password="admin"
10.10.86.31 username="admin" password="admin"

```

## Application Services 3 (AS3)
Here you can find the [AS3 documentation.](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/)

### Install AS3 Extension

Before we can install the AS3 extension at the BIG-IP, we need to download the install package. You can find the extension [here](https://github.com/F5Networks/f5-appsvcs-extension). At the dist path you will find the latest and the Long Time Supported version.

Be aware, that the path will change over time:
```
# cd playbooks/files/

Long Time Supported version:
# wget https://github.com/F5Networks/f5-appsvcs-extension/raw/master/dist/lts/f5-appsvcs-3.5.1-5.noarch.rpm

Latest version:
# wget https://github.com/F5Networks/f5-appsvcs-extension/raw/master/dist/latest/f5-appsvcs-3.9.0-3.noarch.rpm
```

Please adapt the AS3Version variable at the playbook/AS3_install.yaml playbook.

Next you can install the package on the target BIG-IP:

```
# ansible-playbook playbooks/AS3_install.yaml -e target=bigip -e state=present
```

To remove the extension from the BIG-IP, you need to change the state to absent:

```
# ansible-playbook playbooks/AS3_install.yaml -e target=bigip -e state=absent
```


