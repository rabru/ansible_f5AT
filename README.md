# F5 Automation Toolchain demo for Ansible
## Overview

## Install the demo
For this demo I assume that you installed already Ansible on your system. To use the F5 modules on ansible, it is necessary to install additional the following extensions via pip:
...
pip install bigsuds
pip install f5-sdk
...
 I tested the Demo with the following Versions:
...
# python --version
Python 2.7.9
# ansible --version
ansible 2.7.7
# pip show f5-sdk
Name: f5-sdk
Version: 3.0.20
..

If you got git available, the easiest way would be to clone it. An alternative way would be to download the zip file and unpack it on the target system.
Next you need to adapt the settings to your environment.

It is highly recommended to put the password into an ansible vault. To keep this demo as easy as possible I put it into the hosts file.
## Application Services 3 (AS3)
[AS3 documentation:](https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/)


