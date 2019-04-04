BIG-IQ Service Catalog deployment
=================================

This is an example how to deploy a Service Catalog of BIG-IQ CM v5.4. The Service Catalog should be created manually over the BIG-IQ GUI. To get the related json code for the REST call, create an Application manually on the BIG-IQ and use the “View Sample API Request” button to create the related json file.

I stored this file in the template folder and modified it to a jinja2 template.

Example playbook (BIGIQcms.yml):
```
##############################################
# CMS deployment with http and https virtual #
##############################################
#
#vars:
#  - vip: <virtual server ip address>
#  - members:
#    -
#      ip: <pool member ip>
#      name: <node name>
#
---
  - hosts: bigiq
    gather_facts: False
    roles:
      - BIGIQ_AppService
    vars:
      - app_name: "cms2_demo"
      - vip: "10.128.10.27"
      - members:
        -
          ip: "10.10.10.221"
          name: "node1"
        -
          ip: "10.10.10.222"
          name: "node2"

## don't touch: ##
      - j2template: "CMS_deployment"
... 
```

License
-------
Apache V2.0

Author Information
------------------
Ralf Bruenig (r.bruenig@f5.com)

